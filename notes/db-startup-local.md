# 本地开发：后端连 Docker 里的 PostgreSQL

> 个人学习笔记，整理自课堂演示（护花使者）。只放在本 fork 的 `notes/` 分支，不提交回上游仓库。

## 启动顺序

```bash
# 1. 只起数据库（后端跑在宿主机时不需要整个 compose 栈）
docker-compose up -d postgres

# 2. 等它就绪：STATUS 为 healthy 再往下走，否则 seed 会 connection refused
docker-compose ps postgres
docker-compose logs -f postgres   # 需要时看初始化日志

# 3. 准备后端环境变量（读取路径是 backend/.env）
cd backend
cp .env.example .env
```

## 关键配置

`backend/.env.example` 模板里的端口**本来就是 5432**，真正要改的是账号密码，改成与 `docker-compose.yml` 一致：

```dotenv
DATABASE_URL=postgresql://flower_user:flower_password@localhost:5432/flower_guardian
```

## 建表 + 种子数据

```bash
# 在 backend/ 目录、已激活 venv 的前提下执行
python -m app.db.seed

# 再启动后端
uvicorn app.main:app --reload --port 8000
```

`seed.py` 做的事：`Base.metadata.create_all()` 建表，然后插入初始植物数据；库中已有植物时会打印 `Database already has N plants` 并跳过，可重复执行。

## 那个 5555 的坑

不建 `backend/.env` 时，`backend/app/core/config.py` 的默认值是：

```python
DATABASE_URL: str = "postgresql://user:password@localhost:5555/flower_guardian"
```

5555 和 compose 映射的 5432 对不上，接口会连库失败。

## 验证与清理

```bash
# 进 psql 验证
docker-compose exec postgres psql -U flower_user -d flower_guardian -c '\dt'

# 清空数据重建（会删卷，谨慎）
docker-compose down -v
docker-compose up -d postgres
```

## 备注

- README 的「环境变量」表里 `DATABASE_URL` 示例仍写着 5555，与 compose 的 5432 不一致；这是上游现状，本笔记不改上游。
- 之前提给上游的 README 改动（单独启动数据库 / 本地连库说明）已撤回，内容留在本仓库。
