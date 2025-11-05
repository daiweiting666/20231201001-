# 百度贴吧项目E-R图（实体关系图）

## 数据实体定义

### 1. 用户实体 (User)
**主键**: user_id
**属性**:
- user_id (用户ID) - 主键，自增
- username (用户名) - 唯一，非空
- password (密码) - 加密存储
- email (邮箱) - 可选
- avatar (头像) - 文件路径
- signature (签名) - 文本
- registration_time (注册时间) - 时间戳
- last_login_time (最后登录时间) - 时间戳
- status (状态) - 枚举：正常/禁用/删除

### 2. 贴吧实体 (Forum)
**主键**: forum_id
**属性**:
- forum_id (贴吧ID) - 主键，自增
- forum_name (贴吧名称) - 唯一，非空
- description (描述) - 文本
- creator_id (创建者ID) - 外键，关联User
- create_time (创建时间) - 时间戳
- member_count (成员数) - 整数
- post_count (帖子数) - 整数
- status (状态) - 枚举：正常/关闭/审核中

### 3. 帖子实体 (Post)
**主键**: post_id
**属性**:
- post_id (帖子ID) - 主键，自增
- title (标题) - 非空
- content (内容) - 长文本
- author_id (作者ID) - 外键，关联User
- forum_id (所属贴吧ID) - 外键，关联Forum
- create_time (发布时间) - 时间戳
- update_time (更新时间) - 时间戳
- view_count (浏览数) - 整数
- reply_count (回复数) - 整数
- like_count (点赞数) - 整数
- status (状态) - 枚举：正常/删除/审核中/置顶
- is_top (是否置顶) - 布尔值

### 4. 评论实体 (Comment)
**主键**: comment_id
**属性**:
- comment_id (评论ID) - 主键，自增
- content (内容) - 文本
- author_id (作者ID) - 外键，关联User
- post_id (所属帖子ID) - 外键，关联Post
- parent_id (父评论ID) - 外键，关联Comment（自关联）
- create_time (评论时间) - 时间戳
- like_count (点赞数) - 整数
- reply_count (回复数) - 整数
- status (状态) - 枚举：正常/删除/审核中

### 5. 点赞实体 (Like)
**复合主键**: (user_id, target_id, target_type)
**属性**:
- user_id (用户ID) - 外键，关联User
- target_id (目标ID) - 整数
- target_type (目标类型) - 枚举：post/comment
- create_time (点赞时间) - 时间戳

### 6. 收藏实体 (Favorite)
**复合主键**: (user_id, post_id)
**属性**:
- user_id (用户ID) - 外键，关联User
- post_id (帖子ID) - 外键，关联Post
- create_time (收藏时间) - 时间戳

### 7. 关注实体 (Follow)
**复合主键**: (follower_id, followed_id, follow_type)
**属性**:
- follower_id (关注者ID) - 外键，关联User
- followed_id (被关注者ID) - 整数
- follow_type (关注类型) - 枚举：user/forum
- create_time (关注时间) - 时间戳

### 8. 浏览历史实体 (ViewHistory)
**复合主键**: (user_id, post_id, view_time)
**属性**:
- user_id (用户ID) - 外键，关联User
- post_id (帖子ID) - 外键，关联Post
- view_time (浏览时间) - 时间戳
- duration (浏览时长) - 整数（秒）

## 实体关系描述

### 1. 用户关系
- **用户-帖子**: 一对多关系（一个用户可以发布多个帖子）
- **用户-评论**: 一对多关系（一个用户可以发表多个评论）
- **用户-点赞**: 一对多关系（一个用户可以点赞多个帖子/评论）
- **用户-收藏**: 一对多关系（一个用户可以收藏多个帖子）
- **用户-关注**: 自关联多对多关系（用户关注其他用户/贴吧）

### 2. 贴吧关系
- **贴吧-帖子**: 一对多关系（一个贴吧包含多个帖子）
- **贴吧-用户**: 多对多关系（用户关注贴吧）

### 3. 帖子关系
- **帖子-评论**: 一对多关系（一个帖子有多个评论）
- **帖子-点赞**: 一对多关系（一个帖子可以被多个用户点赞）
- **帖子-收藏**: 一对多关系（一个帖子可以被多个用户收藏）
- **帖子-浏览历史**: 一对多关系（一个帖子可以被多个用户浏览）

### 4. 评论关系
- **评论-评论**: 自关联一对多关系（评论可以回复评论）
- **评论-点赞**: 一对多关系（一个评论可以被多个用户点赞）

## E-R图文本表示

```
用户(User) 1 —— N 帖子(Post)
用户(User) 1 —— N 评论(Comment)
用户(User) 1 —— N 点赞(Like)
用户(User) 1 —— N 收藏(Favorite)
用户(User) 1 —— N 浏览历史(ViewHistory)

贴吧(Forum) 1 —— N 帖子(Post)

帖子(Post) 1 —— N 评论(Comment)
帖子(Post) 1 —— N 点赞(Like)
帖子(Post) 1 —— N 收藏(Favorite)
帖子(Post) 1 —— N 浏览历史(ViewHistory)

评论(Comment) 1 —— N 评论(Comment) [自关联]
评论(Comment) 1 —— N 点赞(Like)

用户(User) N —— N 用户(User) [关注关系]
用户(User) N —— N 贴吧(Forum) [关注关系]
```

## 数据库表结构设计

### 用户表 (users)
```sql
CREATE TABLE users (
    user_id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    email VARCHAR(100),
    avatar VARCHAR(255),
    signature TEXT,
    registration_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_login_time TIMESTAMP,
    status ENUM('active', 'disabled', 'deleted') DEFAULT 'active'
);
```

### 贴吧表 (forums)
```sql
CREATE TABLE forums (
    forum_id INT AUTO_INCREMENT PRIMARY KEY,
    forum_name VARCHAR(100) UNIQUE NOT NULL,
    description TEXT,
    creator_id INT,
    create_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    member_count INT DEFAULT 0,
    post_count INT DEFAULT 0,
    status ENUM('active', 'closed', 'pending') DEFAULT 'active',
    FOREIGN KEY (creator_id) REFERENCES users(user_id)
);
```

### 帖子表 (posts)
```sql
CREATE TABLE posts (
    post_id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    content TEXT NOT NULL,
    author_id INT NOT NULL,
    forum_id INT NOT NULL,
    create_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    update_time TIMESTAMP,
    view_count INT DEFAULT 0,
    reply_count INT DEFAULT 0,
    like_count INT DEFAULT 0,
    status ENUM('active', 'deleted', 'pending', 'pinned') DEFAULT 'active',
    is_top BOOLEAN DEFAULT FALSE,
    FOREIGN KEY (author_id) REFERENCES users(user_id),
    FOREIGN KEY (forum_id) REFERENCES forums(forum_id)
);
```

### 评论表 (comments)
```sql
CREATE TABLE comments (
    comment_id INT AUTO_INCREMENT PRIMARY KEY,
    content TEXT NOT NULL,
    author_id INT NOT NULL,
    post_id INT NOT NULL,
    parent_id INT,
    create_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    like_count INT DEFAULT 0,
    reply_count INT DEFAULT 0,
    status ENUM('active', 'deleted', 'pending') DEFAULT 'active',
    FOREIGN KEY (author_id) REFERENCES users(user_id),
    FOREIGN KEY (post_id) REFERENCES posts(post_id),
    FOREIGN KEY (parent_id) REFERENCES comments(comment_id)
);
```

### 点赞表 (likes)
```sql
CREATE TABLE likes (
    user_id INT NOT NULL,
    target_id INT NOT NULL,
    target_type ENUM('post', 'comment') NOT NULL,
    create_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (user_id, target_id, target_type),
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);
```

### 收藏表 (favorites)
```sql
CREATE TABLE favorites (
    user_id INT NOT NULL,
    post_id INT NOT NULL,
    create_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (user_id, post_id),
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    FOREIGN KEY (post_id) REFERENCES posts(post_id)
);
```

### 关注表 (follows)
```sql
CREATE TABLE follows (
    follower_id INT NOT NULL,
    followed_id INT NOT NULL,
    follow_type ENUM('user', 'forum') NOT NULL,
    create_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (follower_id, followed_id, follow_type),
    FOREIGN KEY (follower_id) REFERENCES users(user_id)
);
```

### 浏览历史表 (view_history)
```sql
CREATE TABLE view_history (
    user_id INT NOT NULL,
    post_id INT NOT NULL,
    view_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    duration INT DEFAULT 0,
    PRIMARY KEY (user_id, post_id, view_time),
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    FOREIGN KEY (post_id) REFERENCES posts(post_id)
);
```

## 索引设计

### 主要索引
- users.username (唯一索引)
- users.email (唯一索引)
- posts.author_id (外键索引)
- posts.forum_id (外键索引)
- posts.create_time (时间索引)
- comments.post_id (外键索引)
- comments.parent_id (外键索引)
- likes.target_id_target_type (复合索引)

### 性能优化索引
- posts.status_create_time (复合索引)
- forums.status_member_count (复合索引)
- users.registration_time (时间索引)

## 数据完整性约束

### 业务规则
1. 用户不能重复点赞同一目标
2. 用户不能重复收藏同一帖子
3. 用户不能关注自己
4. 帖子删除时，相关评论和点赞需要级联处理
5. 用户删除时，需要处理相关数据

### 外键约束
- 所有外键关系都设置了级联删除或设置为NULL
- 确保数据的一致性和完整性

---

**E-R图说明**:
- 本E-R图基于百度贴吧需求分析文档V1.0制作
- 展示了项目的完整数据实体关系和数据库设计
- 包含8个主要实体和它们之间的复杂关系
- 提供了详细的数据库表结构和索引设计
- 可作为数据库设计和开发的参考依据