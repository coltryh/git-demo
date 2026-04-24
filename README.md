UPDATE ddmdba.dbo.DDM_PURCHASE_LIST
SET update_time = GETDATE()
OUTPUT inserted.* -- 执行完直接显示改了哪几行
WHERE create_time = (
    -- 3. 在各机构最新的记录中，取出全局最新（时间最大）的那 1 个时间点
    SELECT TOP 1 target_time 
    FROM (
        -- 2. 按机构分组，找出每个机构满足条件的最新记录时间
        SELECT MAX(create_time) AS target_time
        FROM ddmdba.dbo.DDM_PURCHASE_LIST
        -- 1. 你的核心过滤条件全部加在这里：
        WHERE (
            (year = 2025 AND month >= 10) 
            OR year > 2025
        )
        AND submit_flow = -1 -- 这里假设 -1 是取消，请核对你们的取消枚举值
        GROUP BY pharm_eyes_code
    ) AS tmp
    ORDER BY target_time DESC
)
-- 双重保险，防止 update 错数据
AND submit_flow = -1; 
