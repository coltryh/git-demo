package med.acc.utils;

import com.alibaba.excel.EasyExcel;
import com.alibaba.excel.ExcelWriter;
import com.alibaba.excel.annotation.ExcelProperty;
import com.alibaba.excel.write.metadata.WriteSheet;
import com.alibaba.excel.write.metadata.style.WriteCellStyle;
import com.alibaba.excel.write.metadata.style.WriteFont;
import com.alibaba.excel.write.style.HorizontalCellStyleStrategy;
import lombok.extern.slf4j.Slf4j;
import med.acc.pojo.dto.HistoryChangeMailDto;
import org.apache.commons.lang3.ObjectUtils;
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Value;

import javax.mail.MessagingException;
import javax.mail.internet.MimeBodyPart;
import javax.mail.internet.MimeUtility;
import javax.servlet.ServletOutputStream;
import javax.servlet.http.HttpServletResponse;
import java.awt.*;
import java.io.ByteArrayOutputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.lang.reflect.Field;
import java.math.BigDecimal;
import java.math.RoundingMode;
import java.net.URLEncoder;
import java.text.SimpleDateFormat;
import java.time.LocalDate;
import java.time.format.DateTimeFormatter;
import java.util.*;
import java.util.List;
import java.util.stream.Collectors;

@Slf4j
public class FileHandleUtils {
    private static final Logger logger =  LoggerFactory.getLogger(FileHandleUtils.class);

    @Value("${ddm.stc.excel.template.path}")
    private String excelTemplatePath;

    public static void downloadFile(HttpServletResponse response, List<?> list, Class<?> clazz, String tableName, String sheetName, List<String> selectedFields) throws IOException {
        list.removeIf(Objects::isNull);
        if (list.isEmpty()) {
            logger.warn("导出数据为空");
            return;
        }
        //头的策略
        WriteCellStyle headWriteCellStyle = new WriteCellStyle();
        //设置背景为淡绿色
        headWriteCellStyle.setFillForegroundColor(IndexedColors.LIGHT_GREEN.getIndex());
        //设置excel基本布局
        WriteFont headWriteFont = new WriteFont();
        headWriteFont.setFontHeightInPoints((short) 11);
        headWriteFont.setFontName("Arial");
        headWriteCellStyle.setWriteFont(headWriteFont);
        headWriteCellStyle.setHorizontalAlignment(HorizontalAlignment.LEFT);
        //内容设置
        WriteCellStyle contentWriteCellStyle = new WriteCellStyle();
        WriteFont contentWriteFont = new WriteFont();
        //字体大小
        contentWriteFont.setFontHeightInPoints((short) 11);
        contentWriteCellStyle.setWriteFont(contentWriteFont);
        contentWriteCellStyle.setHorizontalAlignment(HorizontalAlignment.LEFT);
        HorizontalCellStyleStrategy styleStrategy = new HorizontalCellStyleStrategy(headWriteCellStyle, contentWriteCellStyle);

        // 文件名
        String fileName = tableName + new SimpleDateFormat("yyyy-MM-dd_HH_mm_ss").format(new Date()) + ".xlsx";
        response.addHeader("Content-Disposition", "attachment;filename=" + URLEncoder.encode(fileName, "UTF-8"));
        response.setContentType("application/vnd.ms-excel;charset=utf-8");

        try (ServletOutputStream outputStream = response.getOutputStream()) {
            ExcelWriter excelWriter;
            WriteSheet writeSheet;

            boolean isDynamicExport = selectedFields != null && !selectedFields.isEmpty();

            if (isDynamicExport) {
                Map<String, String> fieldTitleMap = getFieldTitleMap(clazz, selectedFields);
                List<List<String>> headList = fieldTitleMap.values().stream()
                        .map(Collections::singletonList)
                        .collect(Collectors.toList());

                // ====== 构造数据行 List<List<Object>> ======
                List<List<Object>> dataRows = new ArrayList<>();
                for (Object obj : list) {
                    boolean isLock = false;
                    boolean isConfirmed = false;
                    try {
                        Field field = clazz.getDeclaredField("isLock");
                        field.setAccessible(true);
                        isLock = (boolean) field.get(obj);
                        field = clazz.getDeclaredField("isConfirmed");
                        field.setAccessible(true);
                        isConfirmed = (boolean) field.get(obj);
                    } catch (Exception e) {
                        log.info("obj isn't ProductAdmission Class");
                    }
                    List<Object> row = new ArrayList<>();
                    for (String fieldName : selectedFields) {
                        try {
                            Field field = clazz.getDeclaredField(fieldName);
                            field.setAccessible(true);
                            //对特殊字段的处理
                            if ("lockAndConfirmed".equals(fieldName)){
                                try {
                                    if (!isLock && !isConfirmed) {
                                        row.add(MedConstants.lockConfirmed.NO_LOCK.getValue());
                                    } else if (isLock && !isConfirmed) {
                                        row.add(MedConstants.lockConfirmed.IS_LOCK.getValue());
                                    } else if (!isLock && isConfirmed) {
                                        row.add(MedConstants.lockConfirmed.IS_CONFIRMED.getValue());
                                    } else if (isLock && isConfirmed) {
                                        row.add(MedConstants.lockConfirmed.LOCK_AND_CONFIRMED.getValue());
                                    }
                                } catch (Exception e) {
                                    row.add("");
                                }
                            } else if (Arrays.asList("actualAdmissionDate","unofficialConfirmActualAdmissionDate","infoConfirmActualAdmissionDate","historyActualAdmissionDate").contains(fieldName)) {
                                try {
                                    LocalDate localDate = (LocalDate) field.get(obj);
                                    String formattedDate = localDate.format(DateTimeFormatter.ofPattern("yyyy/MM/dd"));
                                    row.add(formattedDate);
                                } catch (Exception e) {
                                    row.add("");
                                }
                            } else{
                                row.add(field.get(obj));
                            }
                        } catch (NoSuchFieldException | IllegalAccessException e) {
                            row.add(""); // 如果字段获取失败，填空
                        }
                    }
                    dataRows.add(row);
                }

                // 写入器（无类映射）
                excelWriter = EasyExcel.write(outputStream)
                        .head(headList)
                        .registerWriteHandler(styleStrategy)
                        .build();
                writeSheet = EasyExcel.writerSheet(sheetName).build();

                // 分页写入
                int pageSize = 2000;
                int total = dataRows.size();
                for (int i = 0; i < total; i += pageSize) {
                    int toIndex = Math.min(i + pageSize, total);
                    excelWriter.write(dataRows.subList(i, toIndex), writeSheet);
                }

            } else {
                excelWriter = EasyExcel.write(outputStream, clazz)
                        .registerWriteHandler(styleStrategy)
                        .build();
                writeSheet = EasyExcel.writerSheet(sheetName).build();

                int pageSize = 2000;
                int total = list.size();
                for (int i = 0; i < total; i += pageSize) {
                    int toIndex = Math.min(i + pageSize, total);
                    excelWriter.write(list.subList(i, toIndex), writeSheet);
                }
            }

            excelWriter.finish();
        } catch (Exception e) {
            e.printStackTrace();
            throw new RuntimeException("Excel导出失败: " + e.getMessage(), e);
        }
    }

    public static void downloadFile(HttpServletResponse response, List<?> list, Class<?> clazz, String tableName, String sheetName) throws IOException {
        downloadFile(response, list, clazz, tableName, sheetName, null);
    }

    private static Map<String, String> getFieldTitleMap(Class<?> clazz, List<String> selectedFields) {
        Map<String, String> fieldMap = new LinkedHashMap<>();
        Set<String> classFields = Arrays.stream(clazz.getDeclaredFields())
                .map(Field::getName)
                .collect(Collectors.toSet());

        for (String fieldName : selectedFields) {
            if (classFields.contains(fieldName)) {
                try {
                    Field field = clazz.getDeclaredField(fieldName);
                    ExcelProperty prop = field.getAnnotation(ExcelProperty.class);
                    String header = (prop != null && prop.value().length > 0) ? prop.value()[0] : fieldName;
                    fieldMap.put(fieldName, header);
                } catch (NoSuchFieldException e) {
                    fieldMap.put(fieldName, fieldName); // 不太可能出现，因为上面已检查 contains
                }
            } else {
                fieldMap.put(fieldName, fieldName); // 字段不存在，标题用字段名顶上
            }
        }

        return fieldMap;
    }


    public static byte[] generateExcel(List<?> list, Class<?> clazz, String tableName, String sheetName) throws IOException {
        list.removeIf(Objects::isNull);
        if (list.isEmpty()) {
            log.warn("数据为空");
            return null;
        }

        //头的策略
        WriteCellStyle headWriteCellStyle = new WriteCellStyle();
        //设置背景为淡绿色
        headWriteCellStyle.setFillForegroundColor(IndexedColors.LIGHT_GREEN.getIndex());
        //设置excel基本布局
        WriteFont headWriteFont = new WriteFont();
        headWriteFont.setFontHeightInPoints((short) 11);
        headWriteFont.setFontName("Arial");
        headWriteCellStyle.setWriteFont(headWriteFont);
        headWriteCellStyle.setHorizontalAlignment(HorizontalAlignment.LEFT);

//        headWriteCellStyle.setBorderTop(BorderStyle.THIN);
//        headWriteCellStyle.setTopBorderColor(IndexedColors.BLUE_GREY.getIndex());
//        headWriteCellStyle.setBorderBottom(BorderStyle.THIN);
//        headWriteCellStyle.setBottomBorderColor(IndexedColors.BLUE_GREY.getIndex());
//        headWriteCellStyle.setBorderLeft(BorderStyle.THIN);
//        headWriteCellStyle.setLeftBorderColor(IndexedColors.BLUE_GREY.getIndex());
//        headWriteCellStyle.setBorderRight(BorderStyle.THIN);
//        headWriteCellStyle.setRightBorderColor(IndexedColors.BLUE_GREY.getIndex());

        //内容设置
        WriteCellStyle contentWriteCellStyle = new WriteCellStyle();
        WriteFont contentWriteFont = new WriteFont();
        //字体大小
        contentWriteFont.setFontHeightInPoints((short) 11);
        contentWriteCellStyle.setWriteFont(contentWriteFont);
        contentWriteCellStyle.setHorizontalAlignment(HorizontalAlignment.LEFT);
        HorizontalCellStyleStrategy styleStrategy = new HorizontalCellStyleStrategy(headWriteCellStyle, contentWriteCellStyle);

//        contentWriteCellStyle.setBorderTop(BorderStyle.THIN);
//        contentWriteCellStyle.setTopBorderColor(IndexedColors.BLUE_GREY.getIndex());
//        contentWriteCellStyle.setBorderBottom(BorderStyle.THIN);
//        contentWriteCellStyle.setBottomBorderColor(IndexedColors.BLUE_GREY.getIndex());
//        contentWriteCellStyle.setBorderLeft(BorderStyle.THIN);
//        contentWriteCellStyle.setLeftBorderColor(IndexedColors.BLUE_GREY.getIndex());
//        contentWriteCellStyle.setBorderRight(BorderStyle.THIN);
//        contentWriteCellStyle.setRightBorderColor(IndexedColors.BLUE_GREY.getIndex());

        // 文件名
        String fileName = tableName + ".xlsx";

        MimeBodyPart attachPart = new MimeBodyPart();

        ByteArrayOutputStream outputStream = new ByteArrayOutputStream();

        // 对附件文件名进行UTF-8编码
        try {
            attachPart.setFileName(MimeUtility.encodeText(fileName, "UTF-8", "B"));

            ExcelWriter excelWriter;
            WriteSheet writeSheet;

            excelWriter = EasyExcel.write(outputStream, clazz)
                    .registerWriteHandler(styleStrategy)
                    .build();
            writeSheet = EasyExcel.writerSheet(sheetName).build();

            int pageSize = 2000;
            int total = list.size();
            for (int i = 0; i < total; i += pageSize) {
                int toIndex = Math.min(i + pageSize, total);
                excelWriter.write(list.subList(i, toIndex), writeSheet);
            }

            excelWriter.finish();

            byte[] excelBytes = outputStream.toByteArray();
            // 验证文件是否生成成功
            if (ObjectUtils.isEmpty(excelBytes)) {
                log.error("Excel file Error");
                return null;
            }

            return excelBytes;

        } catch (MessagingException e) {
            throw new RuntimeException(e);
        }
    }

}
