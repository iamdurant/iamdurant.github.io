##### spring-boot-data-elasticsearch
**依赖**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
    <version>2.7.6</version>
</dependency>
```

**annotation**
```java
@Repository
@EnableElasticSearchRepositories(basePackage = "")
@Document(indexName = ""， createIndex = true)  标注实体类
@Id  标注主键
@Field(type = FieldType.Text, analyzer = "ik_max_word")  声明字段类型以及其他信息
```

**interface**
```java
@Repository
public interface EsSubjectRepo extends ElasticsearchRepository<EsSubjectInfo, Long> {

}
```

**代码demo**
```java
@Service
public class EsSubjectInfoServiceImpl implements EsSubjectInfoService {
    @Resource
    private EsSubjectRepo repo;

    @Resource
    private ElasticsearchRestTemplate restTemplate;

    @Override
    public void createIndex() {
        IndexOperations indexOps = restTemplate.indexOps(SubjectInfoEs.class);
        indexOps.create();
        Document mapping = indexOps.createMapping(SubjectInfoEs.class);
        indexOps.putMapping(mapping);
    }

    @Override
    public void addDocs() {
        List<SubjectInfoEs> subjectInfoEs = List.of(
                new SubjectInfoEs(1L, "mysql是什么", "mysql是关系型数据库", "wwb", new Date()),
                new SubjectInfoEs(2L, "redis是什么", "redis是基于内存的键值对型数据库", "wwb", new Date()),
                new SubjectInfoEs(3L, "es是什么", "es是搜索引擎", "wwb", new Date()));

        repo.saveAll(subjectInfoEs);
    }

    @Override
    public void find() {
        Iterable<SubjectInfoEs> all = repo.findAll();
        for (SubjectInfoEs subjectInfoEs : all) {
            System.out.println(subjectInfoEs);
        }
    }

    @Override
    public void search() {
        NativeSearchQuery query = new NativeSearchQueryBuilder()
                .withQuery(QueryBuilders.matchQuery("createUser", "wwb"))
                .build();

        SearchHits<SubjectInfoEs> searched = restTemplate.search(query, SubjectInfoEs.class);

        Iterator<SearchHit<SubjectInfoEs>> iterator = searched.stream().iterator();
        while (iterator.hasNext()) {
            SearchHit<SubjectInfoEs> next = iterator.next();
            SubjectInfoEs content = next.getContent();
            System.out.println(content.getSubjectName());
        }
    }
}
```