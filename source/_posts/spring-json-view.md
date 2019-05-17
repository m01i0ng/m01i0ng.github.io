---
title: Spring 中使用 JsonView 控制对象属性显示隐藏
date: 2019-05-17 09:48:59
tags:
  - Spring
  - Java
categories: Java
---

定义 Bean `User.java`：

```java
package tech.newsta.dto;

import com.fasterxml.jackson.annotation.JsonView;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {

    @JsonView(UserSimpleView.class)
    private String username;

    @JsonView(UserDetailView.class)
    private String password;

    public interface UserSimpleView {

    }

    public interface UserDetailView extends UserSimpleView {

    }
}

```

其中 username 指定在请求视图为 `UserSimpleView` 时显示，password 指定在 `UserDetailView` 时显示，同时因为 `UserDetailView` 继承自 `UserSimpleView`，所以前者属性显示的同时会显示后者的所有属性

定义控制器方法如下：

```java
@GetMapping("/user/{id:\\d+}")
@JsonView(User.UserDetailView.class)
public User getInfo(@PathVariable String id) {
    System.out.println(id);

    User user = new User();
    user.setUsername("tom");

    return user;
}
```

在方法上同样使用 `JsonView` 注解指定要获取的视图，即可以此控制字段显示和隐藏
