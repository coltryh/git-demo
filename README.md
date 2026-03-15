 @Override
    public void exportAdmissionOverviewReport(HttpServletResponse response, SearchFormDto searchFormDto) throws IOException {
        // 设置响应头
        response.setContentType("application/vnd.openxmlformats-officedocument.spreadsheetml.sheet");
        response.setCharacterEncoding("utf-8");
        String fileName = "准入概览报表";
        String encodedFileName = URLEncoder.encode(fileName, "UTF-8").replaceAll("\\+", "%20");
        response.setHeader("Content-disposition", "attachment;filename=" + encodedFileName + ".xlsx");

        //创建表头样式
        WriteCellStyle headWriteCellStyle = new WriteCellStyle();
        headWriteCellStyle.setFillForegroundColor(IndexedColors.WHITE.getIndex());
        headWriteCellStyle.setFillPatternType(FillPatternType.SOLID_FOREGROUND);
        headWriteCellStyle.setVerticalAlignment(VerticalAlignment.CENTER);
        headWriteCellStyle.setHorizontalAlignment(HorizontalAlignment.LEFT);

        //模拟Excel默认的细灰网格线
        short gridColor = IndexedColors.GREY_25_PERCENT.getIndex();
        headWriteCellStyle.setBorderBottom(BorderStyle.THIN);
        headWriteCellStyle.setBorderTop(BorderStyle.THIN);
        headWriteCellStyle.setBorderLeft(BorderStyle.THIN);
        headWriteCellStyle.setBorderRight(BorderStyle.THIN);
        headWriteCellStyle.setBottomBorderColor(gridColor);
        headWriteCellStyle.setTopBorderColor(gridColor);
        headWriteCellStyle.setLeftBorderColor(gridColor);
        headWriteCellStyle.setRightBorderColor(gridColor);

        // 字体：11号，取消加粗
        WriteFont headFont = new WriteFont();
        headFont.setFontHeightInPoints((short) 11);
        headFont.setBold(false);
        headWriteCellStyle.setWriteFont(headFont);

        //创建数据基础样式
        WriteCellStyle contentWriteCellStyle = new WriteCellStyle();
        contentWriteCellStyle.setVerticalAlignment(VerticalAlignment.CENTER);

        // 数据行样式
        contentWriteCellStyle.setBorderBottom(BorderStyle.THIN);
        contentWriteCellStyle.setBorderTop(BorderStyle.THIN);
        contentWriteCellStyle.setBorderLeft(BorderStyle.THIN);
        contentWriteCellStyle.setBorderRight(BorderStyle.THIN);
        contentWriteCellStyle.setBottomBorderColor(gridColor);
        contentWriteCellStyle.setTopBorderColor(gridColor);
        contentWriteCellStyle.setLeftBorderColor(gridColor);
        contentWriteCellStyle.setRightBorderColor(gridColor);

        WriteFont contentFont = new WriteFont();
        contentFont.setFontHeightInPoints((short) 11);
        contentWriteCellStyle.setWriteFont(contentFont);

        HorizontalCellStyleStrategy styleStrategy = new HorizontalCellStyleStrategy(headWriteCellStyle, contentWriteCellStyle);

        // 创建Excel写入器
        ExcelWriter excelWriter = EasyExcel.write(response.getOutputStream())
                .registerWriteHandler(styleStrategy)
                .registerWriteHandler(new LongestMatchColumnWidthStyleStrategy())
                .registerWriteHandler(new CellWriteHandler() {
                    // 只需要缓存数据行的样式即可，表头已经全局搞定了
                    private CellStyle contentLeftStyle;
                    private CellStyle contentRightStyle;

                    @Override
                    public void afterCellDispose(CellWriteHandlerContext context) {
                        // 【关键修改】：表头直接跳过，不在这里处理对齐
                        if (context.getHead()) {
                            return;
                        }

                        Cell cell = context.getCell();
                        Workbook workbook = context.getWriteWorkbookHolder().getWorkbook();
                        int colIndex = context.getColumnIndex();

                        // 数据行的对齐逻辑
                        if (colIndex == 0) {
                            // 第一列左对齐
                            if (contentLeftStyle == null) {
                                contentLeftStyle = workbook.createCellStyle();
                                contentLeftStyle.cloneStyleFrom(cell.getCellStyle());
                                contentLeftStyle.setAlignment(HorizontalAlignment.LEFT);
                            }
                            cell.setCellStyle(contentLeftStyle);
                        } else {
                            // 其他列右对齐
                            if (contentRightStyle == null) {
                                contentRightStyle = workbook.createCellStyle();
                                contentRightStyle.cloneStyleFrom(cell.getCellStyle());
                                contentRightStyle.setAlignment(HorizontalAlignment.RIGHT);
                            }
                            cell.setCellStyle(contentRightStyle);
                        }
                    }
                })
                .build();

        // ========== Sheet1: 项目维度 ==========
        List<String> projectMetrics = Arrays.asList("正式准入", "非正式准入", "未准入");
        Map<String, AdmissionOverviewDto> projectData = getProjectOverallAdmissionStatisticsReport(searchFormDto);
        if (projectData != null && !projectData.isEmpty()) {
            List<List<String>> head = buildSimpleHead("整体准入统计(项目)", projectMetrics);
            List<List<Object>> dataList = buildProjectDataNew(projectData, projectMetrics);
            WriteSheet projectSheet = EasyExcel.writerSheet(0, "项目维度").head(head).autoTrim(true).build();
            excelWriter.write(dataList, projectSheet);
        }

        // ========== Sheet2: 月度维度 ==========
        List<String> monthMetrics = Arrays.asList("正式准入", "非正式准入");
        Map<String, AdmissionOverviewDto> monthData = getMonthOverallAdmissionStatisticsReport(searchFormDto);
        if (monthData != null && !monthData.isEmpty()) {
            List<List<String>> head = buildSimpleHead("整体准入统计(月份)", monthMetrics);
            List<List<Object>> dataList = buildMonthDataNew(monthData, monthMetrics);
            WriteSheet monthSheet = EasyExcel.writerSheet(1, "月度维度").head(head).autoTrim(true).build();
            excelWriter.write(dataList, monthSheet);
        }

        // ========== Sheet3: 医院级别维度 ==========
        List<String> terminalLevelMetrics = Arrays.asList("正式准入", "非正式准入", "准入中", "总计", "完成进度");
        Map<String, AdmissionOverviewDto> terminalLevelData = getTerminalLevelOverallAdmissionStatisticsReport(searchFormDto);
        if (terminalLevelData != null && !terminalLevelData.isEmpty()) {
            List<List<String>> head = buildSimpleHead("整体准入统计(医院级别)", terminalLevelMetrics);
            List<List<Object>> dataList = buildFullData(terminalLevelData, terminalLevelMetrics);
            WriteSheet terminalLevelSheet = EasyExcel.writerSheet(2, "医院级别维度").head(head).autoTrim(true).build();
            excelWriter.write(dataList, terminalLevelSheet);
        }

        // ========== Sheet4: 销售区域维度 ==========
        List<String> districtMetrics = Arrays.asList("正式准入", "非正式准入", "准入中", "总计", "完成进度");
        Map<String, Map<String, AdmissionOverviewDto>> districtData = getDistrictOverallAdmissionStatisticsReport(searchFormDto);
        if (districtData != null && !districtData.isEmpty()) {
            List<List<String>> head = buildSimpleHead("整体准入统计(销售区域)", districtMetrics);
            List<List<Object>> dataList = buildDistrictFullData(districtData, districtMetrics);
            WriteSheet districtSheet = EasyExcel.writerSheet(3, "销售区域维度").head(head).autoTrim(true).build();
            excelWriter.write(dataList, districtSheet);
        }

        // ========== Sheet5: 商务大区维度 ==========
        List<String> regionMetrics = Arrays.asList("正式准入", "非正式准入", "准入中", "总计", "完成进度");
        Map<String, AdmissionOverviewDto> regionData = getTerminalRegionOverallAdmissionStatisticsReport(searchFormDto);
        if (regionData != null && !regionData.isEmpty()) {
            List<List<String>> head = buildSimpleHead("整体准入统计(商务大区)", regionMetrics);
            List<List<Object>> dataList = buildFullData(regionData, regionMetrics);
            WriteSheet regionSheet = EasyExcel.writerSheet(4, "商务大区维度").head(head).autoTrim(true).build();
            excelWriter.write(dataList, regionSheet);
        }

        // ========== Sheet6: 省份维度 ==========
        List<String> provinceMetrics = Arrays.asList("正式准入", "非正式准入", "实际准入汇总", "未准入", "目标数量", "完成进度");
        Map<String, AdmissionOverviewDto> provinceData = getTerminalProvinceOverallAdmissionStatisticsReport(searchFormDto);
        if (provinceData != null && !provinceData.isEmpty()) {
            List<List<String>> head = buildSimpleHead("整体准入统计(省份)", provinceMetrics);
            List<List<Object>> dataList = buildFullDetailData(provinceData, provinceMetrics);
            WriteSheet provinceSheet = EasyExcel.writerSheet(5, "省份维度").head(head).autoTrim(true).build();
            excelWriter.write(dataList, provinceSheet);
        }

        // ========== Sheet7: 商务一线维度 ==========
        List<String> managerMetrics = Arrays.asList("正式准入", "非正式准入", "实际准入汇总", "目标数量", "完成进度");
        Map<String, AdmissionOverviewDto> managerData = getManagerOverallAdmissionStatisticsReport(searchFormDto);
        if (managerData != null && !managerData.isEmpty()) {
            List<List<String>> head = buildSimpleHead("商务一线", managerMetrics);
            List<List<Object>> dataList = buildManagerDistributorData(managerData, managerMetrics);
            WriteSheet managerSheet = EasyExcel.writerSheet(6, "商务一线维度").head(head).autoTrim(true).build();
            excelWriter.write(dataList, managerSheet);
        }

        // ========== Sheet8: 经销商维度 ==========
        List<String> distributorMetrics = Arrays.asList("正式准入", "非正式准入", "实际准入汇总", "未准入", "目标数量", "完成进度");
        Map<String, AdmissionOverviewDto> distributorData = getDistributorOverallAdmissionStatisticsReport(searchFormDto);
        if (distributorData != null && !distributorData.isEmpty()) {
            List<List<String>> head = buildSimpleHead("经销商", distributorMetrics);
            List<List<Object>> dataList = buildFullDetailData(distributorData, distributorMetrics);
            WriteSheet distributorSheet = EasyExcel.writerSheet(7, "经销商维度").head(head).autoTrim(true).build();
            excelWriter.write(dataList, distributorSheet);
        }

        excelWriter.finish();
    }

    /**
     * 构建项目数据（正式准入、非正式准入、未准入）
     */
    private List<List<Object>> buildProjectDataNew(Map<String, AdmissionOverviewDto> dataMap, List<String> metrics) {
        List<List<Object>> dataList = new ArrayList<>();
        for (Map.Entry<String, AdmissionOverviewDto> entry : dataMap.entrySet()) {
            List<Object> row = new ArrayList<>();
            row.add(entry.getKey());
            AdmissionOverviewDto dto = entry.getValue();
            row.add(dto.getFormalNum());
            row.add(dto.getNotFormalNum());
            row.add(dto.getNotAccessibleNum());
            dataList.add(row);
        }
        return dataList;
    }

    /**
     * 构建月份数据（3个字段）
     */
    private List<List<Object>> buildMonthDataNew(Map<String, AdmissionOverviewDto> dataMap, List<String> metrics) {
        List<List<Object>> dataList = new ArrayList<>();
        for (Map.Entry<String, AdmissionOverviewDto> entry : dataMap.entrySet()) {
            List<Object> row = new ArrayList<>();
            row.add(entry.getKey());
            AdmissionOverviewDto dto = entry.getValue();
            row.add(dto.getFormalNum());
            row.add(dto.getNotFormalNum());
            dataList.add(row);
        }
        return dataList;
    }

    /**
     * 构建完整详细数据（6个字段）
     */
    private List<List<Object>> buildFullDetailData(Map<String, AdmissionOverviewDto> dataMap, List<String> metrics) {
        List<List<Object>> dataList = new ArrayList<>();
        for (Map.Entry<String, AdmissionOverviewDto> entry : dataMap.entrySet()) {
            List<Object> row = new ArrayList<>();
            row.add(entry.getKey());
            AdmissionOverviewDto dto = entry.getValue();
            row.add(dto.getFormalNum());
            row.add(dto.getNotFormalNum());
            row.add(dto.getAdmissionSummaryNum());
            row.add(dto.getNotAccessibleNum());
            row.add(dto.getTotalNum());
            row.add(dto.getAdmissionProgress());
            dataList.add(row);
        }
        return dataList;
    }

    /**
     * 构建商务一线/经销商数据（5个字段：正式准入、非正式准入、实际准入汇总、目标数量、完成进度）
     */
    private List<List<Object>> buildManagerDistributorData(Map<String, AdmissionOverviewDto> dataMap, List<String> metrics) {
        List<List<Object>> dataList = new ArrayList<>();
        for (Map.Entry<String, AdmissionOverviewDto> entry : dataMap.entrySet()) {
            List<Object> row = new ArrayList<>();
            row.add(entry.getKey());
            AdmissionOverviewDto dto = entry.getValue();
            row.add(dto.getFormalNum());
            row.add(dto.getNotFormalNum());
            row.add(dto.getAdmissionSummaryNum());
            row.add(dto.getTotalNum());
            row.add(dto.getAdmissionProgress());
            dataList.add(row);
        }
        return dataList;
    }

    /**
     * 构建完整数据（5个字段：正式准入、非正式准入、准入中、总计、完成进度）
     */
    private List<List<Object>> buildFullData(Map<String, AdmissionOverviewDto> dataMap, List<String> metrics) {
        List<List<Object>> dataList = new ArrayList<>();
        for (Map.Entry<String, AdmissionOverviewDto> entry : dataMap.entrySet()) {
            List<Object> row = new ArrayList<>();
            row.add(entry.getKey());
            AdmissionOverviewDto dto = entry.getValue();
            row.add(dto.getFormalNum());
            row.add(dto.getNotFormalNum());
            row.add(dto.getAdmissionSummaryNum() - dto.getFormalNum() - dto.getNotFormalNum());
            row.add(dto.getTotalNum());
            row.add(dto.getAdmissionProgress());
            dataList.add(row);
        }
        return dataList;
    }

    /**
     * 构建销售区域完整数据（5个字段）
     */
    private List<List<Object>> buildDistrictFullData(Map<String, Map<String, AdmissionOverviewDto>> districtData, List<String> metrics) {
        List<List<Object>> dataList = new ArrayList<>();
        for (Map.Entry<String, Map<String, AdmissionOverviewDto>> entry : districtData.entrySet()) {
            String projectName = entry.getKey();
            Map<String, AdmissionOverviewDto> regionMap = entry.getValue();
            for (Map.Entry<String, AdmissionOverviewDto> regionEntry : regionMap.entrySet()) {
                List<Object> row = new ArrayList<>();
                row.add(projectName + " - " + regionEntry.getKey());
                AdmissionOverviewDto dto = regionEntry.getValue();
                row.add(dto.getFormalNum());
                row.add(dto.getNotFormalNum());
                row.add(dto.getAdmissionSummaryNum() - dto.getFormalNum() - dto.getNotFormalNum());
                row.add(dto.getTotalNum());
                row.add(dto.getAdmissionProgress());
                dataList.add(row);
            }
        }
        return dataList;
    }

    /**
     * 构建简单表头（单行）
     */
    private List<List<String>> buildSimpleHead(String dimensionName, List<String> metrics) {
        List<List<String>> head = new ArrayList<>();
        head.add(Arrays.asList(dimensionName));
        for (String metric : metrics) {
            head.add(Arrays.asList(metric));
        }
        return head;
    }

    /**
     * 构建简单数据（Map结构）
     */
    private List<List<Object>> buildSimpleData(Map<String, AdmissionOverviewDto> dataMap, String keyField, List<String> metrics) {
        List<List<Object>> dataList = new ArrayList<>();
        for (Map.Entry<String, AdmissionOverviewDto> entry : dataMap.entrySet()) {
            List<Object> row = new ArrayList<>();
            row.add(entry.getKey());
            AdmissionOverviewDto dto = entry.getValue();
            row.add(dto.getFormalNum());
            row.add(dto.getNotFormalNum());
            row.add(dto.getAdmissionSummaryNum());
            row.add(dto.getNotAccessibleNum());
            row.add(dto.getTotalNum());
            row.add(dto.getFormalProgress());
            row.add(dto.getAdmissionProgress());
            dataList.add(row);
        }
        return dataList;
    }

    /**
     * 构建简单数据（List结构）
     */
    private List<List<Object>> buildSimpleDataFromList(List<Map<String, AdmissionOverviewDto>> dataListMap, List<String> metrics) {
        List<List<Object>> dataList = new ArrayList<>();
        for (Map<String, AdmissionOverviewDto> item : dataListMap) {
            for (Map.Entry<String, AdmissionOverviewDto> entry : item.entrySet()) {
                List<Object> row = new ArrayList<>();
                row.add(entry.getKey());
                AdmissionOverviewDto dto = entry.getValue();
                row.add(dto.getFormalNum());
                row.add(dto.getNotFormalNum());
                row.add(dto.getAdmissionSummaryNum());
                row.add(dto.getNotAccessibleNum());
                row.add(dto.getTotalNum());
                row.add(dto.getFormalProgress());
                row.add(dto.getAdmissionProgress());
                dataList.add(row);
            }
        }
        return dataList;
    }

    /**
     * 构建销售区域数据（嵌套Map结构）
     */
    private List<List<Object>> buildDistrictData(Map<String, Map<String, AdmissionOverviewDto>> districtData, List<String> metrics) {
        List<List<Object>> dataList = new ArrayList<>();
        for (Map.Entry<String, Map<String, AdmissionOverviewDto>> entry : districtData.entrySet()) {
            String projectName = entry.getKey();
            Map<String, AdmissionOverviewDto> regionMap = entry.getValue();
            for (Map.Entry<String, AdmissionOverviewDto> regionEntry : regionMap.entrySet()) {
                List<Object> row = new ArrayList<>();
                row.add(projectName + " - " + regionEntry.getKey());
                AdmissionOverviewDto dto = regionEntry.getValue();
                row.add(dto.getFormalNum());
                row.add(dto.getNotFormalNum());
                row.add(dto.getNotAccessibleNum());
                dataList.add(row);
            }
        }
        return dataList;
    }
