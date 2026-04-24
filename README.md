UPDATE DDM_PURCHASE_LIST
SET update_time = GETDATE()  -- SQL Server 获取当前时间的函数
WHERE create_time = (
    -- 第三层：在所有机构的最新取消记录中，取出全局“最年轻”的一个时间
    SELECT TOP 1 target_time 
    FROM (
        -- 第二层：按机构分组，找出每个机构最近一个月内的最新取消时间
        SELECT MAX(create_time) AS target_time
        FROM DDM_PURCHASE_LIST
        -- 条件1：最近一个月内 (SQL Server 的时间倒推写法)
        WHERE create_time >= DATEADD(month, -1, GETDATE())
          -- 条件2：状态为取消
          AND submit_flow = -1 
        GROUP BY pharm_eyes_code
    ) AS tmp
    ORDER BY target_time DESC
)
-- 加上这个条件，防止刚好有两条记录 create_time 一模一样，误改了非取消的单子
AND submit_flow = -1;
