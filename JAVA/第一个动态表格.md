# 第一个动态表格

## 功能介绍

一个带有登录的极简用户后台（含分页）

## 所需文件

| 模块     | 文件名            | 用处                        |
| -------- | ----------------- | --------------------------- |
| 查询工具 | com.util.JDBCUtil | 实现数据库连接              |
|  |  |  |
| 登录    | com.domain.User   | 根据用户表创建的类          |
| 登录    | com.Dao.user      | 根据传入数据进行指定SQL查询 |
| 登录    | com.servlet.login | 登录控制器                  |
| 登录 | web/login.jsp | 登录视图 |
|  |  |  |
| 数据展示 | com.domain.Student | 根据用户表创建的类          |
| 数据展示 | com.Dao.Student | 根据传入数据进行指定SQL查询 |
| 数据展示 | com.servlet.student | 控制器                  |
| 数据展示 | web/student.jsp | 视图 |
|  |  |  |
|  分页| com.domain.DataPage | 用于数据展示时返回的对象 |



## 主要还是servlet和jsp

```java
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    	// 获取参数
        String page = request.getParameter("page");
        String row = request.getParameter("row");
    	// 参数设置默认值
        page = page == null ? "1" : page;
        row = row == null ? "50" : row;
    	// 获取分页的对象
        DataPage student = StudentDao.getStudent(page,row);
    	// 设置标题
        String title = "标题";
    	// 将返回的数据从列表中取出
        List list = student.getList();
        int totalCount = student.getTotalCount();
        int totalPage = student.getTotalPage();
    	// 发送到视图
        request.setAttribute("list",list);                  // 数据
        request.setAttribute("title",title);                // 标题
        request.setAttribute("totalCount",totalCount);      // 总数据数
        request.setAttribute("totalPage",totalPage);        // 总页数
        request.setAttribute("page",page);                  // 当前页数
        request.setAttribute("row",row);                  // 当前页数
    	/**
    	* getRequestDispatcher()包含两个方法，分别是请求转发和请求包含。
    	* 请求转发： rd.forward( request , response );
		* 请求包含： rd.include( request  , response);
    	*/
        request.getRequestDispatcher("/student.jsp").forward(request,response);
    }
```



```jsp
<table border="1px">
    <tr>
        <td>姓名</td>
        <td>邮箱</td>
        <td>密码</td>
        <td>真实姓名</td>
        <td>性别</td>
        <td>生日</td>
    </tr>
    <c:forEach items="${list}" var="student">
        <tr>
            <td>${student.username}</td>
            <td>${student.email}</td>
            <td>${student.password}</td>
            <td>${student.realname}</td>
            <td>${student.sex}</td>
            <td>${student.birthday}</td>
        </tr>
    </c:forEach>
</table>

共查找出${totalCount}条，${totalPage}页<br/>
<c:if test="${page-1>0}">
<a href="${pageContext.request.contextPath}?page=${page-1}&row=${row}">上一页</a><br/>
</c:if>
<a href="${pageContext.request.contextPath}?page=${page+1}&row=${row}">下一页</a><br/>
```

