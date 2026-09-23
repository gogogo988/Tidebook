这是一个专为商业垂钓船长设计的网站，用于管理日程安排、预订及钓友。
目前的 Windows 本地部署版 v1.1.0，采用 React 前端＋Node.js 后端＋SQLite 数据库。一个 Node.js 服务同时提供网页和接口，适合在个人电脑上试用、修改和部署。

1. 整体架构

浏览器负责显示船期、表单和弹窗；后端负责保存数据、核实身份、控制权限和报名名额。Cloudflare 通道负责让外部钓友访问电脑上的服务，数据库仍保存在你的电脑上。

2. 前端技术栈

技术	用途
React 19	船期列表、船长工作台、报名及订金弹窗
TypeScript	页面和业务数据的类型定义
Vite	开发及生产打包
Tailwind CSS＋自定义 CSS	页面布局、配色和手机适配
shadcn 风格组件、Radix UI	按钮、表格、对话框、侧栏等
Lucide React、Sonner	图标及操作结果提示
浏览器 Fetch API	调用后端接口

主要界面是一个单页应用：切换船期、预约、统计等内容时，通常不重新加载整个网页。最新的钓友流程通过同一个弹窗切换注册、报名和订金提交。

账户设置页另用 HTML、CSS 和原生 JavaScript 实现。虽然源码保留了 app/page.tsx 等目录名称，以及部分原项目依赖，当前运行方式是 Vite＋React，不依赖 Next.js 服务端运行。

3. 后端技术栈

技术	用途
Node.js 24 或以上	运行网站服务
Node 原生 HTTP 模块	提供网页和 API，无 Express/NestJS
Node 内置 SQLite 模块	保存账户、船期、预约等记录
Node 文件系统模块	保存订金凭证、钓获照片和备份
Node Crypto／scrypt	密码哈希及会话令牌处理
PowerShell＋CMD	Windows 初始化、启动和维护

主要接口：

接口	职责
/api/auth/*	注册、登录、用户名修改、密码修改及退出
/api/data	船期、报名、订金状态、取消、结算、船长权限等
/api/file	上传及读取图片，并检查访问权限
/health	检查服务是否正常运行

报名容量、收费结算和船长权限都由后端校验，不会仅依赖前端按钮是否可见。报名占位采用带条件的数据库写入，避免多人同时报名时超出容量。

银行付款目前采用“线下转账 → 提交凭证 → 船长确认到账”，没有接入自动扣款或银行流水接口。

4. 源码构成

以解压后的 Tidebook-Windows 为根目录：

目录或文件	内容与修改用途
source/app/page.tsx	主要页面、导航、船期卡片、报名弹窗及船长工作台
source/app/globals.css	全局样式、布局及手机适配
source/components/ui/	可复用界面组件
source/lib/pricing.ts	前端价格展示辅助逻辑
source/entry.tsx	React 启动入口
source/vite.config.ts	前端构建配置
source/drizzle/*.sql	数据库建表及迁移脚本
server/server.mjs	HTTP 服务、路由、认证、会话和静态网页服务
server/runtime.mjs	SQLite 初始化、数据库访问及文件存储
server/security.mjs	密码校验、哈希和令牌工具
server/data.ts	核心业务：船期、报名、取消、收款状态等
server/pricing.ts	按人数收费的校验和最终价格计算
server/captains.ts	两位船长的权限管理与转交
server/file.ts	图片上传和访问控制
server/account.html/js/css	独立账户设置页面
server/admin.mjs	初始化船长、重置密码和备份
tools/	构建、通道、自启动及 Windows 辅助脚本
tests/smoke.mjs	接口集成测试
web/	编译产物，浏览器实际加载的网页
data/	运行后生成的真实数据库和图片
backups/	手动备份结果

其中 server/data.ts、pricing.ts、captains.ts、file.ts 会编译成同名 .mjs 文件供 Node.js 执行。修改这些业务逻辑时应编辑 .ts，再重新构建；其余手写 .mjs 文件可直接修改。

5. 自己修改时怎么入手

改页面文字、报名流程：source/app/page.tsx
改颜色、字号、布局：source/app/globals.css
改收费规则：server/pricing.ts，并同步前端展示
改报名、取消及收款逻辑：server/data.ts
改账户和密码规则：server/server.mjs、server/security.mjs
改外网连接方式：tools/tunnel.mjs

首次准备开发环境，在 source 目录安装依赖；之后在项目根目录执行：

node tools/build.mjs

构建会更新后端生成文件和 web 网页。日常启动网站不需要重新安装前端依赖。

目前代码以便于本机运行和集中修改为主，主页面和业务接口较集中。后续功能增多时，可以进一步拆成独立的船期、预约、账户和统计模块。
