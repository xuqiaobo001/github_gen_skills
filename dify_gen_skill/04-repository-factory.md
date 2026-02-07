# Skill 4: `repository-factory` — Repository Factory 数据访问层

**适用场景**: 需要支持多种数据存储后端、或需要在测试中轻松替换数据层的项目。

**Dify 源码引用**:

- `api/core/rag/datasource/vdb/` — 向量数据库 Repository 实现（Weaviate / Qdrant / Milvus 等 15+）
- `api/core/storage/` — 文件存储 Repository（S3 / Azure / Local 等）
- `api/factories/` — Factory 类根据配置动态选择实现

**核心代码骨架**:

```python
# 1. 定义抽象基类
class BaseVectorDB(ABC):
    @abstractmethod
    def create_collection(self, name: str, dimension: int) -> None: ...

    @abstractmethod
    def add_texts(self, texts: list[Document], **kwargs) -> list[str]: ...

    @abstractmethod
    def search_by_vector(self, query_vector: list[float], top_k: int) -> list[Document]: ...

    @abstractmethod
    def delete_by_ids(self, ids: list[str]) -> None: ...

# 2. 具体实现
class QdrantVector(BaseVectorDB):
    def __init__(self, collection_name: str, config: QdrantConfig):
        self._client = QdrantClient(url=config.url, api_key=config.api_key)
    # ... 实现所有抽象方法

# 3. Factory 根据配置动态实例化
class VectorFactory:
    @staticmethod
    def get_vector_db(vector_type: str, collection_name: str) -> BaseVectorDB:
        match vector_type:
            case "qdrant":
                return QdrantVector(collection_name, qdrant_config)
            case "weaviate":
                return WeaviateVector(collection_name, weaviate_config)
            case _:
                raise ValueError(f"Unsupported vector type: {vector_type}")
```

**关键规则**:

- 基类定义完整 CRUD 接口，子类必须实现所有抽象方法
- Factory 通过配置字符串（非硬编码）选择实现，便于运行时切换
- Service 层通过构造函数注入 Repository，不直接 `import` 具体实现
- 测试时可注入 Mock Repository，无需启动真实数据库

**反模式**:

- ❌ Service 层直接 `from qdrant import QdrantClient` 耦合具体实现
- ❌ Factory 中使用 `if/elif` 长链而不用 `match/case` 或注册表模式
- ❌ 基类接口不完整，导致子类各自添加非标准方法
