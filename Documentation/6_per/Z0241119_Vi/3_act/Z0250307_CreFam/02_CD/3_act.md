## Z026/04/22（三）

### Q：通过shell，将CSV文件导入到Oracle db table，要对CSV文件做检查，非空，单项目，重复等检查，不正确的记录生成新文件，正确的导入table，实现

### A：Deep seek

这个需求可以拆解为：**CSV预处理（校验） + 正确数据导入 + 错误数据导出**。下面给出一个完整的Shell脚本实现方案，使用`awk`进行高效校验，`sqlldr`（SQL*Loader）进行批量导入。

---

## 一、整体设计思路

```
CSV文件
    │
    ▼
┌─────────────────────────────────────┐
│  Step 1: 校验                        │
│  - 非空检查（关键字段）                │
│  - 单项目检查（字段数量）              │
│  - 重复检查（根据主键/关键列）         │
└─────────────────────────────────────┘
    │                    │
    ▼                    ▼
  正确记录              错误记录
    │                    │
    ▼                    ▼
┌──────────┐      ┌──────────┐
│ sqlldr   │      │ 错误文件  │
│ 导入table │      │(.bad/.err)│
└──────────┘      └──────────┘
```

---

## 二、完整脚本实现

```bash
#!/bin/bash
# ============================================================================
# 脚本名称: csv_to_oracle.sh
# 功能描述: 校验CSV文件并将正确数据导入Oracle表，错误数据导出到单独文件
# 使用方法: ./csv_to_oracle.sh <input.csv> <table_name> <key_column>
# ============================================================================

set -euo pipefail

# ==================== 配置区（请根据实际情况修改） ====================
ORACLE_SID="your_sid"
ORACLE_USER="your_username"
ORACLE_PASS="your_password"
ORACLE_TABLE=""      # 从参数获取
CSV_INPUT=""         # 从参数获取
KEY_COLUMN=""        # 从参数获取（用于重复检查的列，如ID）

# Oracle SQL*Loader 控制文件路径
CTL_FILE="./load_data.ctl"
LOG_DIR="./logs"
BAD_FILE_DIR="./bad_records"

# CSV字段分隔符（默认逗号）
DELIMITER=","
# 期望的字段数（请根据实际表结构修改）
EXPECTED_FIELDS=5

# ==================== 函数定义 ====================

# 显示使用帮助
usage() {
    echo "Usage: $0 <csv_file> <table_name> <key_column>"
    echo "Example: $0 ./data.csv my_table id"
    exit 1
}

# 检查依赖
check_dependencies() {
    command -v sqlldr >/dev/null 2>&1 || { echo "Error: sqlldr not found"; exit 1; }
    command -v awk >/dev/null 2>&1 || { echo "Error: awk not found"; exit 1; }
}

# 创建必要的目录
create_dirs() {
    mkdir -p "$LOG_DIR" "$BAD_FILE_DIR"
}

# 校验CSV文件：非空、字段数、重复
validate_csv() {
    local input_file=$1
    local valid_file=$2
    local invalid_file=$3
    
    echo "正在校验文件: $input_file"
    
    # 使用awk进行多条件校验
    awk -v delim="$DELIMITER" \
        -v expected="$EXPECTED_FIELDS" \
        -v key_col="$KEY_COLUMN" \
        -v valid="$valid_file" \
        -v invalid="$invalid_file" '
    BEGIN {
        FS = delim
        OFS = FS
        # 用于重复检查的关联数组
        delete seen
        valid_count = 0
        invalid_count = 0
    }
    {
        line_num = NR
        is_valid = 1
        error_msg = ""
        
        # 1. 字段数量检查
        if (NF != expected) {
            is_valid = 0
            error_msg = error_msg "字段数不正确(期望" expected "，实际" NF ");"
        }
        
        # 2. 关键字段非空检查（假设第一列是主键列）
        if (key_col != "" && $key_col ~ /^[[:space:]]*$/) {
            is_valid = 0
            error_msg = error_msg "关键字段列" key_col "为空;"
        }
        
        # 3. 非空检查（所有字段不能为空，可根据需求调整）
        for (i = 1; i <= NF; i++) {
            if ($i ~ /^[[:space:]]*$/) {
                is_valid = 0
                error_msg = error_msg "字段" i "为空;"
            }
        }
        
        # 4. 重复检查（基于关键列）
        if (key_col != "") {
            key_value = $key_col
            if (key_value in seen) {
                is_valid = 0
                error_msg = error_msg "关键值'" key_value "'重复(首次出现行:" seen[key_value] ");"
            } else {
                seen[key_value] = line_num
            }
        }
        
        # 输出到对应文件
        if (is_valid) {
            print $0 >> valid
            valid_count++
        } else {
            # 错误记录：原始行 + 分隔符 + 错误信息
            print $0 delim error_msg >> invalid
            invalid_count++
        }
    }
    END {
        print "校验完成: 正确记录=" valid_count ", 错误记录=" invalid_count
        print valid_count, invalid_count
    }' "$input_file"
    
    return $?
}

# 使用SQL*Loader导入正确数据
load_to_oracle() {
    local data_file=$1
    local table_name=$2
    
    echo "正在导入数据到表: $table_name"
    
    # 动态生成控制文件
    cat > "$CTL_FILE" << EOF
LOAD DATA
INFILE '$data_file'
BADFILE '$LOG_DIR/${table_name}.bad'
DISCARDFILE '$LOG_DIR/${table_name}.dsc'
APPEND
INTO TABLE $table_name
FIELDS TERMINATED BY '$DELIMITER'
OPTIONALLY ENCLOSED BY '"'
TRAILING NULLCOLS
(
    -- 请根据实际表结构修改字段映射
    col1,
    col2,
    col3,
    col4,
    col5
)
EOF

    # 执行sqlldr
    sqlldr $ORACLE_USER/$ORACLE_PASS@$ORACLE_SID \
        control="$CTL_FILE" \
        log="$LOG_DIR/${table_name}_load.log" \
        errors=1000
    
    if [ $? -eq 0 ]; then
        echo "导入成功！"
    else
        echo "导入过程中出现错误，请检查日志: $LOG_DIR/${table_name}_load.log"
        return 1
    fi
}

# ==================== 主流程 ====================

main() {
    # 参数检查
    if [ $# -lt 3 ]; then
        usage
    fi
    
    CSV_INPUT="$1"
    ORACLE_TABLE="$2"
    KEY_COLUMN="$3"
    
    # 检查输入文件是否存在
    if [ ! -f "$CSV_INPUT" ]; then
        echo "Error: 输入文件 $CSV_INPUT 不存在"
        exit 1
    fi
    
    # 准备输出文件名
    timestamp=$(date +"%Y%m%d_%H%M%S")
    base_name=$(basename "$CSV_INPUT" .csv)
    VALID_FILE="${base_name}_valid_${timestamp}.csv"
    INVALID_FILE="${BAD_FILE_DIR}/${base_name}_invalid_${timestamp}.csv"
    
    echo "=========================================="
    echo "CSV导入Oracle开始"
    echo "输入文件: $CSV_INPUT"
    echo "目标表: $ORACLE_TABLE"
    echo "=========================================="
    
    # 1. 检查依赖和创建目录
    check_dependencies
    create_dirs
    
    # 2. 校验CSV
    validate_csv "$CSV_INPUT" "$VALID_FILE" "$INVALID_FILE"
    
    # 3. 检查是否有正确记录
    valid_count=$(wc -l < "$VALID_FILE" 2>/dev/null || echo "0")
    if [ "$valid_count" -gt 0 ]; then
        # 4. 导入正确数据
        load_to_oracle "$VALID_FILE" "$ORACLE_TABLE"
    else
        echo "没有正确记录，跳过导入"
    fi
    
    # 5. 输出结果汇总
    echo "=========================================="
    echo "处理完成"
    echo "正确记录文件: $VALID_FILE"
    echo "错误记录文件: $INVALID_FILE"
    echo "日志目录: $LOG_DIR"
    echo "=========================================="
}

# 执行主函数
main "$@"
```

---

## 三、使用说明

### 1. 修改配置

根据你的环境修改脚本开头的配置区：

```bash
ORACLE_SID="your_sid"        # Oracle实例名
ORACLE_USER="your_username"  # 数据库用户名
ORACLE_PASS="your_password"  # 数据库密码
EXPECTED_FIELDS=5            # CSV期望的字段数
DELIMITER=","                # CSV分隔符
```

### 2. 调整SQL*Loader控制文件

根据你的表结构修改控制文件中的字段映射：

```sql
(
    col1,     -- 对应CSV第1列
    col2,     -- 对应CSV第2列
    col3,     -- 对应CSV第3列
    col4,     -- 对应CSV第4列
    col5      -- 对应CSV第5列
)
```

如果需要跳过某些列，可以使用`FILLER`：

```sql
(
    col1,
    col2,
    skip_col FILLER,   -- 跳过CSV第3列
    col3
)
```

### 3. 运行脚本

```bash
# 赋予执行权限
chmod +x csv_to_oracle.sh

# 执行
./csv_to_oracle.sh ./data.csv my_table 1
# 参数1: CSV文件路径
# 参数2: 目标表名
# 参数3: 用于重复检查的关键列序号（1表示第1列）
```

---

## 四、输出文件说明

| 文件类型 | 路径 | 说明 |
|---------|------|------|
| 正确记录 | `./正确数据_时间戳.csv` | 通过校验的记录 |
| 错误记录 | `./bad_records/错误数据_时间戳.csv` | 未通过校验的记录，每行末尾追加错误原因 |
| 导入日志 | `./logs/表名_load.log` | SQL*Loader导入日志 |
| 坏文件 | `./logs/表名.bad` | SQL*Loader拒绝的记录 |
| 丢弃文件 | `./logs/表名.dsc` | SQL*Loader丢弃的记录 |

---

## 五、扩展建议

### 1. 更灵活的非空检查

如果只需要检查特定字段非空，修改awk中的非空检查逻辑：

```awk
# 只检查第1、3、5列
required_fields = "1,3,5"
split(required_fields, req_arr, ",")
for (i in req_arr) {
    col = req_arr[i]
    if ($col ~ /^[[:space:]]*$/) {
        is_valid = 0
        error_msg = error_msg "必填字段" col "为空;"
    }
}
```

### 2. 支持日期格式校验

```awk
# 检查第2列是否为有效日期（YYYYMMDD格式）
if ($2 !~ /^[0-9]{8}$/) {
    is_valid = 0
    error_msg = error_msg "日期格式错误;"
}
```

### 3. 直接使用外部表（无需sqlldr）

如果Oracle支持外部表，可以创建外部表直接查询CSV，然后通过SQL进行校验和插入。

---

这个方案将校验和导入解耦，便于调试和监控。根据你的实际表结构和业务规则，可以灵活调整校验逻辑。
