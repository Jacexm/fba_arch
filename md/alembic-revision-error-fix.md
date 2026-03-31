# 修复 fba alembic revision 报错问题

## 问题

在当前 uv 环境中执行 `fba alembic revision` 命令时报错：

```
asyncpg.exceptions.InvalidCatalogNameError: database "fba" does not exist
```

## 原因

数据库配置不匹配：

| 配置项 | 值 | 来源 |
|--------|-----|------|
| 默认数据库名 | `fba` | `backend/core/conf.py` 中 `DATABASE_SCHEMA: str = 'fba'` |
| 实际数据库名 | `fba_db` | `docker-compose.local.yml` 中 `POSTGRES_DB: fba_db` |

Alembic 在创建 revision 时会根据配置连接数据库，由于 `.env` 文件中没有显式配置 `DATABASE_SCHEMA`，程序使用了默认值 `fba`，但 Docker 中实际创建的数据库名为 `fba_db`，导致连接失败。

## 解决方法

在 `backend/.env` 文件中添加数据库配置：

```bash
DATABASE_SCHEMA='fba_db'
```

修改后的 `.env` 文件相关部分：

```bash
# Database
DATABASE_TYPE='postgresql'
DATABASE_HOST='127.0.0.1'
DATABASE_PORT=25432
DATABASE_USER='postgres'
DATABASE_PASSWORD='postgres716'
DATABASE_SCHEMA='fba_db'
```

## 验证

执行 `fba alembic revision` 命令，成功生成迁移文件：

```
✓ 迁移文件生成成功
```

## 建议

1. 确保 `.env.example` 文件中包含所有必要的配置项，避免新开发者遗漏配置
2. 本地开发环境配置文件（`.env`）应与 Docker 配置保持一致
