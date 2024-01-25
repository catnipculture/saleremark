# 项目介绍

- 企业的销售要进行培训，由技术人员进行辅导并考评检测培训效果，所以有了这个小系统。
- 实现了系统的登录验证、请求拦截验证、基础模块（用户管理、角色管理、销售管理）、业务模块（评分管理、评分结果）。
- 除了基本的CRUD之外，其中评分结果模块实现了数据可视化及图表的联动。

主要角色有管理员、销售、评委



# 环境要求

1.运行环境：最好是java jdk1.8,我们在这个平台上运行的。其他版本理论上也可以。 

2.IDE环境：IDEA,Eclipse,Myeclipse都可以。推荐IDEA; 

3.tomcat环境：Tomcat7.x,8.X,9.x版本均可 

4.硬件环境：windows7/8/10 4G内存以上；或者Mac OS; 

5.是否Maven项目：是；查看源码目录中是否包含pom.xml;若包含，则为maven项目，否则为非maven.项目 

6.数据库：MySql5.7/8.0等版本均可；

# 技术栈

- 

# 技术框架：jQuery + MySQL5.7 + mybatis + shiro + Layui + HTML + CSS + JS + jpa

1.使用Navicati或者其它工具，在mysql中创建对应sq文件名称的数据库，并导入项目的sql文件； 

2.使用IDEA/Eclipse/MyEclipse导入项目，修改配置，运行项目； 

3.将项目中config-propertiesi配置文件中的数据库配置改为自己的配置，然后运行；

# 运行指导

idea导入源码空间站顶目教程说明(Vindows版)-ssm篇：

http://mtw.so/5MHvZq 

源码看好后直接在网站付款下单即可，付款成功会自动弹出百度网盘链接，网站地址：[http://codegym.top](http://codegym.top/)。 

其它问题请关注公众号：**IT小舟**,关注后发送消息即可，都会给您回复的。若没有及时回复请耐心等待，通常当天会有回复

# 运行截图

## 界面![QQ截图20240113113107](https://gulimallcativen.oss-cn-shenzhen.aliyuncs.com/bishe/QQ%E6%88%AA%E5%9B%BE20240113113107.png)

![QQ截图20240113113148](https://gulimallcativen.oss-cn-shenzhen.aliyuncs.com/bishe/QQ%E6%88%AA%E5%9B%BE20240113113148.png)

![QQ截图20240113113155](https://gulimallcativen.oss-cn-shenzhen.aliyuncs.com/bishe/QQ%E6%88%AA%E5%9B%BE20240113113155.png)

![QQ截图20240113113206](https://gulimallcativen.oss-cn-shenzhen.aliyuncs.com/bishe/QQ%E6%88%AA%E5%9B%BE20240113113206.png)

![QQ截图20240113113217](https://gulimallcativen.oss-cn-shenzhen.aliyuncs.com/bishe/QQ%E6%88%AA%E5%9B%BE20240113113217.png)

![QQ截图20240113113225](https://gulimallcativen.oss-cn-shenzhen.aliyuncs.com/bishe/QQ%E6%88%AA%E5%9B%BE20240113113225.png)



# 代码

UserController

```java
package cn.temptation.web;

import cn.temptation.dao.RoleDao;
import cn.temptation.dao.UserDao;
import cn.temptation.domain.Role;
import cn.temptation.domain.User;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;
import org.springframework.data.jpa.domain.Specification;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.persistence.criteria.*;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

@Controller
public class UserController {
    @Autowired
    private UserDao userDao;
    @Autowired
    private RoleDao roleDao;

    @RequestMapping("/user")
    public String index() {
        return "user";
    }

    @RequestMapping("/user_list")
    @ResponseBody
    public Map<String, Object> userList(@RequestParam Map<String, Object> queryParams) {
        Map<String, Object> result = new HashMap<>();

        try {
            Integer page = Integer.parseInt(queryParams.get("page").toString());
            Integer limit = Integer.parseInt(queryParams.get("limit").toString());
            String condition = (String) queryParams.get("condition");
            String keyword = (String) queryParams.get("keyword");

            // 创建查询规格对象
            Specification<User> specification = (Root<User> root, CriteriaQuery<?> query, CriteriaBuilder cb) -> {
                Predicate predicate = null;
                Path path = null;

                if (condition != null && !"".equals(condition) && keyword != null && !"".equals(keyword)) {
                    switch (condition) {
                        case "username":    // 用户名称
                            path = root.get("username");
                            predicate = cb.like(path, "%" + keyword + "%");
                            break;
                        case "rolename":    // 角色名称
                            path = root.join("role", JoinType.INNER);
                            predicate = cb.like(path.get("rolename"), "%" + keyword + "%");
                            break;
                    }
                }

                return predicate;
            };

            Pageable pageable = PageRequest.of(page - 1, limit, Sort.Direction.ASC, "userid");

            Page<User> users = userDao.findAll(specification, pageable);

            result.put("code", 0);
            result.put("msg", "查询OK");
            result.put("count", users.getTotalElements());
            result.put("data", users.getContent());
        } catch (Exception e) {
            e.printStackTrace();
            result.put("code", 500);
            result.put("msg", "服务器内部错误");
            result.put("count", 0);
            result.put("data", new ArrayList());
        }

        return result;
    }

    @RequestMapping("/user_delete")
    @ResponseBody
    public Integer userDelete(@RequestParam String userid) {
        try {
            userDao.deleteById(Integer.parseInt(userid));
            return 0;
        } catch (Exception e) {
            e.printStackTrace();
        }

        return -1;
    }

    @RequestMapping("/user_view")
    public String view(Integer userid, Model model) {
        User user = new User();
        if (userid != null) {
            user = userDao.getOne(userid);
        }
        model.addAttribute("user", user);
        return "user_view";
    }

    @RequestMapping("/role_load")
    @ResponseBody
    public List<Role> roleList() {
        return roleDao.findAll();
    }

    @RequestMapping("/user_update")
    @ResponseBody
    public Integer userUpdate(User user) {
        try {
            userDao.save(user);
            return 0;
        } catch (Exception e) {
            e.printStackTrace();
        }

        return -1;
    }
}
```





```java
package cn.temptation.web;

import cn.temptation.dao.SalesDao;
import cn.temptation.dao.ScoreDao;
import cn.temptation.domain.Sales;
import cn.temptation.domain.Score;
import cn.temptation.domain.User;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;
import org.springframework.data.jpa.domain.Specification;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.persistence.criteria.*;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

@Controller
public class ScoreController {
    @Autowired
    private ScoreDao scoreDao;
    @Autowired
    private SalesDao salesDao;

    @RequestMapping("/score")
    public String index() {
        return "score";
    }

    @RequestMapping("/score_list")
    @ResponseBody
    public Map<String, Object> scoreList(@RequestParam Map<String, Object> queryParams, HttpServletRequest request) {
        Map<String, Object> result = new HashMap<>();

        try {
            HttpSession session = request.getSession();
            User user = (User) session.getAttribute("user");

            Integer page = Integer.parseInt(queryParams.get("page").toString());
            Integer limit = Integer.parseInt(queryParams.get("limit").toString());
            String condition = (String) queryParams.get("condition");
            String keyword = (String) queryParams.get("keyword");

            // 创建查询规格对象
            Specification<Score> specification = (Root<Score> root, CriteriaQuery<?> query, CriteriaBuilder cb) -> {
                Predicate predicate = null;
                Path path = null;

                // 从session中取出当前用户信息，获取该用户创建的数据
                if (null != user) {
                    path = root.join("user", JoinType.INNER);
                    predicate = cb.equal(path.get("userid"), user.getUserid());
                }

                if (condition != null && !"".equals(condition) && keyword != null && !"".equals(keyword)) {
                    switch (condition) {
                        case "salesname":    // 销售名称
                            path = root.join("sales", JoinType.INNER);
                            predicate = cb.like(path.get("salesname"), "%" + keyword + "%");
                            break;
                    }
                }

                return predicate;
            };

            Pageable pageable = PageRequest.of(page - 1, limit, Sort.Direction.ASC, "scoreid");

            Page<Score> scores = scoreDao.findAll(specification, pageable);

            result.put("code", 0);
            result.put("msg", "查询OK");
            result.put("count", scores.getTotalElements());
            result.put("data", scores.getContent());
        } catch (Exception e) {
            e.printStackTrace();
            result.put("code", 500);
            result.put("msg", "服务器内部错误");
            result.put("count", 0);
            result.put("data", new ArrayList());
        }

        return result;
    }

    @RequestMapping("/score_delete")
    @ResponseBody
    public Integer scoreDelete(@RequestParam String scoreid) {
        try {
            scoreDao.deleteById(Integer.parseInt(scoreid));
            return 0;
        } catch (Exception e) {
            e.printStackTrace();
        }

        return -1;
    }

    @RequestMapping("/score_view")
    public String view(Integer scoreid, Model model) {
        Score score = new Score();
        if (scoreid != null) {
            score = scoreDao.findByScoreid(scoreid);
        }
        model.addAttribute("score", score);
        return "score_view";
    }

    @RequestMapping("/sales_load")
    @ResponseBody
    public List<Sales> salesList() {
        return salesDao.findAll();
    }

    @RequestMapping("/score_update")
    @ResponseBody
    public Integer scoreUpdate(Score score, HttpServletRequest request) {
        try {
            HttpSession session = request.getSession();
            User user = (User) session.getAttribute("user");

            if (null != user) {
                score.setUser(user);
            }

            scoreDao.save(score);
            return 0;
        } catch (Exception e) {
            e.printStackTrace();
        }

        return -1;
    }
}
```

