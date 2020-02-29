## 实体类

首先创建两个实体类，**PageRequest**于前台接收来的分页参数（**当前页数**和**每页显示数量**）、**PageResult**用于返回前端所需要的参数（**数据总数**、**总页数**、**返回数据**）

```java
package com.lksun.library.entity.Page;

public class PageRequest {
    // 当前页数 默认为1
    private int pageNum = 1;

    // 每页显示的数量 默认为5
    private int pageSize = 5;
		
    // 用于计算初始数据
    private int offSet;

    public int getPageNum() {
        return pageNum;
    }
  
    public void setPageNum(int pageNum) {
        this.pageNum = pageNum;
    }
  
    public int getPageSize() {
        return pageSize;
    }

    public void setPageSize(int pageSize) {
        this.pageSize = pageSize;
    }

    public int getOffSet() {
        // 此处计算  (当前页数-1)*每页显示数量
        return (this.pageNum - 1) * this.pageSize;
    }

    public void setOffSet(int offSet) {
        this.offSet = offSet;
    }

    @Override
    public String toString() {
        return "PageRequest{" +
                "pageNum=" + pageNum +
                ", pageSize=" + pageSize +
                ", offSet=" + offSet +
                '}';
    }
}

```





```java
package com.lksun.library.entity.Page;

import java.util.List;

public class PageResult {

    // 当前页码
    private Integer pageNum;

    // 每页数量
    private Integer pageSize;

    // 数据总条数
    private Integer totalSize;

    // 总页数
    private Integer totalPages;

    // 返回数据
    private List<?> content;

    /**
     * @param content 返回的数据
     * @param pageRequest  分页类
     * @param totalSize 数据的总条数
     */
    public PageResult(List<?> content, PageRequest pageRequest, Integer totalSize){
        this.content = content;
        this.totalSize = totalSize;
        this.pageSize = pageRequest.getPageSize();
        this.pageNum = pageRequest.getPageNum();
        // 总数÷每页数量 向上取整并强转为int得到页数
        this.totalPages = (int) Math.ceil(this.totalSize / this.pageSize);  
    }

    public Integer getPageNum() {
        return pageNum;
    }

    public void setPageNum(Integer pageNum) {
        this.pageNum = pageNum;
    }

    public Integer getPageSize() {
        return pageSize;
    }

    public void setPageSize(Integer pageSize) {
        this.pageSize = pageSize;
    }

    public Integer getTotalSize() {
        return totalSize;
    }

    public void setTotalSize(Integer totalSize) {
        this.totalSize = totalSize;
    }

    public int getTotalPages() {
        return totalPages;
    }

    public void setTotalPages(Integer totalPages) {
        this.totalPages = totalPages;
    }

    public List<?> getContent() {
        return content;
    }

    public void setContent(List<?> content) {
        this.content = content;
    }

    @Override
    public String toString() {
        return "PageResult{" +
                "pageNum=" + pageNum +
                ", pageSize=" + pageSize +
                ", totalSize=" + totalSize +
                ", totalPages=" + totalPages +
                ", content=" + content +
                '}';
    }
}

```





## Mapper

```java
@Repository
//@Mapper  // 如果配置了MapperScan则不需要此注解了
public interface ReaderMapper {
		// 获取所有
    List<Reader> get(PageRequest pageRequest);

  	// 获取总数
    Integer getCount();
}

```





## 业务层

### ReaderServiceImpl

```java
@Service
public class ReaderServiceImpl implements ReaderService {

    @Autowired
    private ReaderMapper readerMapper;

    @Override
    public List<Reader> get(PageRequest pageRequest) {
        return readerMapper.get(pageRequest);
    }

    public Integer getCount(){
        return readerMapper.getCount();
    }
}

```



### ReaderServiceImpl

```java
@Service
public class ReaderServiceImpl implements ReaderService {

    @Autowired
    private ReaderMapper readerMapper;

    @Override
    public List<Reader> get(PageRequest pageRequest) {
        return readerMapper.get(pageRequest);
    }

    public Integer getCount(){
        return readerMapper.getCount();
    }
}

```





## Mapping

```xml
<?xml version="1.0" encoding="UTF-8"?>
        <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- 命名空间与Mapper文件对应 -->
<mapper namespace="com.lksun.library.Mapper.ReaderMapper">
		
    <select id="getCount" resultType="java.lang.Integer">
        select count(*) from readers
    </select>
  
    <select id="get" resultType="Reader">
      	<!-- 使用的是实体类计算好的属性-->
        select * from readers LIMIT #{offSet},#{pageSize}
        <!-- LIMIT (#{pageNum}-1)*#{pageSize},#{pageSize};  错误 -->
        <!-- LIMIT ${(pageNum-1)*pageSize},${pageSize};  正确-->
    </select>

</mapper>

```





### 控制器

```java
@Controller("ReadsIndexController")
@RequestMapping("/reader")
public class IndexController {

    @Autowired
    private  ReaderService readerService;

    @RequestMapping(value = "",method = RequestMethod.GET)
    public String list(PageRequest pageRequest, Map<String,Object> map){
        // 首先获取所有数据
        List<Reader> readerList = readerService.get(pageRequest);
        // 实例化分页结果的类 
        PageResult pageResult = new PageResult(readerList,pageRequest,readerService.getCount());
        map.put("list",readerList);
        return "/reader/index";
    }

}
```

