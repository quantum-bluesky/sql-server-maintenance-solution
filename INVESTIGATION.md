# Báo cáo điều tra: Ola Hallengren Index Optimize + Update Statistics (SQL Server 2022)

## Phạm vi
.....
## Tham chiếu (repo cục bộ)
- `README.md` xác nhận SQL Server 2022 được hỗ trợ và liệt kê các script bảo trì, gồm `IndexOptimize.sql`.  ([README.md L14-25](README.md#L14-25))
- `IndexOptimize.sql` chứa khai báo thủ tục, tham số mặc định, quy tắc kiểm tra tham số, và cách dựng lệnh `UPDATE STATISTICS`.  ([IndexOptimize.sql L11-47](IndexOptimize.sql#L11-47))

## Tham chiếu ngoài
- DeepWiki (SQL Server Maintenance Solution Overview) dùng để tham khảo cấu trúc và trang mục lục của tài liệu ngoài repo. {https://deepwiki.com/olahallengren/sql-server-maintenance-solution/}

## Kết quả điều tra
### 1) Hỗ trợ SQL Server 2022
`README.md` liệt kê SQL Server 2022 trong danh sách version tương thích.  ([README.md L24-25](README.md#L24-25))

### 2) Thủ tục bảo trì index & statistics
`IndexOptimize` là stored procedure xử lý index/statistics. Các tham số chính cho Optimize Index + Update Statistics:
- Ngưỡng/loại thao tác fragmentation:
  - `@FragmentationMedium` mặc định `INDEX_REORGANIZE,INDEX_REBUILD_ONLINE,INDEX_REBUILD_OFFLINE`
  - `@FragmentationHigh` mặc định `INDEX_REBUILD_ONLINE,INDEX_REBUILD_OFFLINE`  ([IndexOptimize.sql L15-16](IndexOptimize.sql#L15-16))
- Tùy chọn thống kê:
  - `@UpdateStatistics`, `@OnlyModifiedStatistics`, `@StatisticsModificationLevel`
  - `@StatisticsSample`, `@StatisticsResample`  ([IndexOptimize.sql L26-30](IndexOptimize.sql#L26-30))
- Kiểm soát chạy: `@TimeLimit`, `@Delay`, `@LogToTable`, `@MinNumberOfPages`  ([IndexOptimize.sql L19-36](IndexOptimize.sql#L19-36))

### 3) Quy tắc kiểm tra tham số thống kê
Thủ tục có các ràng buộc:
- `@UpdateStatistics` chỉ nhận `ALL`, `COLUMNS`, hoặc `INDEX`.  ([IndexOptimize.sql L897-901](IndexOptimize.sql#L897-901))
- `@OnlyModifiedStatistics` và `@StatisticsModificationLevel` loại trừ nhau.  ([IndexOptimize.sql L921-925](IndexOptimize.sql#L921-925))
- `@StatisticsSample` phải trong khoảng 1–100 nếu có.  ([IndexOptimize.sql L929-933](IndexOptimize.sql#L929-933))
- `@StatisticsResample` chỉ nhận `Y/N` và không được dùng cùng `@StatisticsSample`.  ([IndexOptimize.sql L937-946](IndexOptimize.sql#L937-946))

### 4) Cách dựng lệnh UPDATE STATISTICS
Khi đủ điều kiện, thủ tục dựng và chạy:
```
UPDATE STATISTICS [schema].[object] [statistics_name] WITH <options>
```
Các option hỗ trợ gồm:
- `FULLSCAN` khi `@StatisticsSample = 100`
- `SAMPLE <n> PERCENT` khi `@StatisticsSample` khác 100
- `NORECOMPUTE` nếu statistic có `no-recompute`
- `RESAMPLE` khi được yêu cầu  ([IndexOptimize.sql L2280-2318](IndexOptimize.sql#L2280-2318))

Nếu dùng incremental statistics trên bảng partitioned, lệnh có thể thêm:
```
ON PARTITIONS(<partition_number>)
```
 ([IndexOptimize.sql L2347-2351](IndexOptimize.sql#L2347-2351))

## Ghi chú
- Tham khảo `MaintenancePlan/HUONG_DAN_SU_DUNG.md` và `MaintenancePlan/LOGIC_FLOW.md`
- Tham khảo DeepWiki {https://deepwiki.com/olahallengren/sql-server-maintenance-solution/}
