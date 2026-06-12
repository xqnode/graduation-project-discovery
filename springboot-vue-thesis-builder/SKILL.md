---
name: springboot-vue-thesis-builder
description: >-
  Builds Spring Boot 3.2 + Vue 3 + MySQL capstone/thesis projects from one-sentence prompts.
  Session auth, AdminLayout UI, Result/PageResult API, single base.sql. Works in empty repos
  and any coding agent. Paste full file as system prompt, rules, CLAUDE.md, or AGENTS.md.
disable-model-invocation: true
---

# Spring Boot + Vue 毕设生成器

单 MD 自包含。将**全文**贴入 Agent 的 system / rules / 项目说明，用户消息只写业务需求即可。

## 默认行为（一句话自主）

- 用户一句业务描述 → 输出简短执行计划 → **立即**按 W0–W7 写代码，**不等确认**（除非用户说「先别写代码」）
- 模块 **3–5 个**；过大则自动缩减；假设写入 `docs/02-详细设计.md`
- 需求极模糊时自行补全并标注假设，仍收窄到 3–5 模块后再写代码
- 需读写文件 + shell；中断后从未完成波次续跑

## 执行波次

每波结束标记 `[x] Wn`；本波失败修完再继续。

| 波次 | 任务 | 完成标准 |
|------|------|----------|
| W0 | 脚手架 | 空仓生成 §文件清单；有仓跳过 |
| W1 | 库表 | `sql/base.sql` + init-db 脚本 |
| W2 | 后端 | 全层 + `mvn -q -DskipTests compile` |
| W3 | 管理端 | api + *List + router + 侧栏；§UI门禁 |
| W4 | 用户端 | FrontLayout 业务页（无则跳过） |
| W5 | 控制台 | stats API + Dashboard 扩展 |
| W6 | 文档 | README + docs/01 + docs/02 + 部署 |
| W7 | 验证 | `npm run build` + §交付自检 |

## 硬约束（技术 + 后端 + API + 库）

**栈：** Java21 · Boot3.2 · MyBatis-Plus · HttpSession · Hutool · Vue3 script-setup · Vite5173 · Pinia · ElementPlus · Lucide · MySQL8 `base_db` · 后端9090 · 前端5173 代理 `/api` `/uploads`。

**禁：** JWT · Spring Security · Swagger · 动态菜单 · RBAC表 · dto分包 · 逻辑删除 · `sql/incremental/`。

**包名** `com.base`。**分层** Controller→Service/Impl→Mapper→Entity。**命名** `biz_book`→`Book`→`BookExt`→`/api/books`。

**API：** `Result{code,message,data}` code=200；分页 `PageResult` current从1；管理端 `authHelper.requireAdmin(session)`。

**Session：** 键 `loginUserId`；放行 `/api/auth/login|register`。**角色** ADMIN/USER。

**库表：** 只改 `sql/base.sql`；必有 `id/create_time/update_time`(VARCHAR19)；保留 `sys_user` 种子 admin/admin、aaa/123、bbb/123。

**双端：** ADMIN→AdminLayout(`/dashboard` `/users` `/profile/*`)；USER→FrontLayout(`/user/home` `/user/profile/*`)；路由 `meta.admin|front` 互斥。

## W0 文件清单

`README` `.gitignore` `sql/base.sql` `scripts/init-db.*` `springboot/pom.xml` `application.yml` `com/base/**`(BaseApplication, common, exception, entity User/UserExt/DashboardStats, mapper, service, controller Auth/User/Profile/Dashboard/File) `vue/package.json` `vite.config.js` `index.html` `src/main.js` `App.vue` `utils/*` `store/*` `api/*` `router` `styles/{theme,global,auth}.css` `layout/{Admin,Front}Layout` `components/{AppLogo,UserAvatar,AvatarUpload,FrontFooter}` `views/{Login,Register,Dashboard,UserList,Profile,ChangePassword}` `views/user/Home.vue`

**W0 过大时分两轮：** W0a 后端+sql → `mvn compile`；W0b 前端壳 → `npm run build`。

## W0 卡点（必写，漏则跑不通）

- **application.yml：** `cors allow-credentials: true`；`multipart max-file-size: 2MB`；`app.upload.path: ./uploads`；库名 `base_db`
- **vite.config.js：** proxy `/api`、`/uploads` → `http://localhost:9090`
- **BaseApplication：** `PaginationInnerInterceptor` + `MetaObjectHandler`(Hutool `DateUtil.now` 填 createTime/updateTime) + `LoginInterceptor` + 静态映射 `/uploads/**`
- **LoginInterceptor：** Session 键 `loginUserId`；未登录返回 401 JSON
- **request.js：** `withCredentials: true`；`code===200` 取 `data`
- **router 守卫：** `meta.admin|front` 隔离；未登录跳 `/login`
- **auth.css 必选 class：** `auth-page` `auth-bg` `orb` `orb--1|2|3` `grid-overlay` `auth-stage` `auth-card` `auth-header` `auth-form` `auth-btn`（Login/Register 模板 class 与之一致）
- **index.html：** 引入 DM Sans + Noto Sans SC 字体

## UI 一致性（三锁一克隆）

1. **锁色** — primary `#2563eb` sidebar `#0f172a` bg `#f1f5f9`；Element primary 同步；写入 `theme.css`
2. **锁壳** — W0 先生成 `styles/`、`AdminLayout`、`FrontLayout`、`Login`、`Register`、`Dashboard`、`UserList`；AdminLayout 侧栏220px暗色；Login 必须 orb+毛玻璃 auth-card（禁白底简登录）；FrontLayout 顶栏 grid max1200
3. **锁克隆** — 业务 CRUD **整文件复制 UserList.vue** 只改列/表单/API；禁换布局/第二套UI库

**列表页结构：** `.page-container>.content-card>.toolbar-row(搜/重置/新增)>el-table(rowIndex)>footer pagination(small)>el-dialog(480px)`

**W3 UI门禁：** primary/sidebar色对 · Login非简版 · 侧栏非Element默认白 · 业务List从UserList克隆 · 无Tailwind/AntDesign

## 编译验证（W7）

```bash
cd springboot && mvn -q -DskipTests compile
cd vue && npm install && npm run build
```

## 用户 Prompt 示例

> 做一个校园二手书交易平台毕设：用户发书下单，管理员管分类和订单。

## 交付自检

W0–W6完成 · base.sql含种子 · 无JWT等禁项 · UI门禁通过 · compile/build通过 · admin/aaa可演示 · 无TODO占位
