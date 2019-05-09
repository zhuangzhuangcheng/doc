## 常见的Restful Api 设计

为了方便不同的前端设备与后端进行通信，需要统一的API设计机制，所以使用 [RESTful API](http://en.wikipedia.org/wiki/Representational_state_transfer) 作为我们的API设计规范。

详细参考：

> [RESTful API 设计指南](http://www.ruanyifeng.com/blog/2014/05/restful_api.html?bsh_bid=516759003)

此文档列出一些常见的API设计，供以参考，不正确之处还需修正。下面以`employee(员工)`为例，及相关对象：`org(机构)`等，展示在`spring mvc`下设计api接口。

#### 新增

使用`Post`请求方式

?> **POST** http://localhost:8080/employees

```java
@PostMapping("/employees")
public ResultVO add(@RequestBody @Validated Employee emp) {
    employeeService.insert(emp);
    return ResultVO.success("新增员工成功！");
}
```

#### 批量新增

使用`Post`请求方式

?> **POST** http://localhost:8080/employees/batch

``` java
@PostMapping("/employees/batch")
public ResultVO add(@RequestBody @Validated EmployeeBatchAddRequest emps) {
    employeeService.insertBatch(emps.getList());
    return ResultVO.success("批量新增成功！");
}

@Data
public class EmployeeBatchAdd {
    @NotEmpty(message = "集合不能为空")
    @Valid
    private List<Employee> list;
}
```

#### 修改

使用`Put`请求方式

?> **PUT** http://localhost:8080/employees

``` java
@PutMapping("/employees")
public ResultVO update(@RequestBody @Validated Employee emp) {
    employeeService.updateById(emp);
    return ResultVO.success("修改成功！");
}
```

#### 批量修改

使用`Put`请求方式

?> **PUT** http://localhost:8080/employees/batch

``` java
@PutMapping("/employees/batch")
public ResultVO batchUpdate(@RequestBody @Validated EmployeeBatchUpdate emps) {
    employeeService.insertBatch(emps.getList());
    return ResultVO.success("批量修改成功！");
}

@Data
public class EmployeeBatchUpdate {
    @NotEmpty(message = "集合不能为空！")
    @Valid
    private List<Employee> list;
}
```

#### 删除

使用`Delete`请求方式

?> **DELETE** http://localhost:8080/employees/{id}

``` java
@DeleteMapping("/{id}")
public ResultVO delete(@PathVariable @NotBlank(message = "id不能为空") String id) {
    employeeService.deleteById(id);
    return ResultVO.success("删除成功！");
}
```

#### 批量删除

使用`Delete`请求方式，用请求体传输删除的id集合数据

?> **DELETE** http://localhost:8080/employees

```java
@DeleteMapping
public ResultVO batchDel(@RequestBody @NotEmpty(message = "数据不能为空") List<String> ids) {
    employeeService.deleteBatchIds(ids);
    return ResultVO.success("批量删除成功！");
}
```

