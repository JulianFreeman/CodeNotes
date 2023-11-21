# 学习 Next.js

欢迎来到 Next.js 应用路由教程！在这个免费的交互式课程中，你将会通过构建一个全栈网页应用来学习 Next.js 的主要功能。

> 译注：当然本翻译文件不可能为交互式的啦~

## 我们要构建什么

![dashboard](learn_nextjs_images/dashboard.avif)

在本教程中，我们将会构建一个简化版本的金融仪表盘，并拥有如下功能：

- 一个公共的主页面
- 一个登录页面
- 受身份验证保护的仪表盘页面
- 用户可以添加、编辑和删除单据

这个仪表盘也会附带一个数据库，我们会在[之后的章节](#第六章-设置数据库)中设置。

学完本教程后，你会基本掌握开发全栈 Next.js 应用所需的主要技能。

## 概览

下面是我们要在本教程中学习的功能：

- **样式**：在 Next.js 中设置样式的不同方式。
- **优化**：如何优化图片、链接和字体。
- **路由**：如何使用文件系统路由创建嵌套的层和页面。
- **数据抓取**：如何在 Vercel 上设置数据库，如何高效抓取和流式传输。
- **检索和分页**：如何使用 URL 搜索参数实现检索和分页。
- **更改数据**：如何使用 React 服务器操作更改数据，以及重新验证 Next.js 缓存。
- **错误处理**：如何处理一般的和 `404` 错误。
- **表单验证和访问**：如何做服务器端的表单验证以及提高可访问性的技巧。
- **身份验证**：如何使用 [NextAuth.js](https://next-auth.js.org/) 和中间件为你的应用添加身份验证。
- **元数据**：如何添加元数据以及为你的应用的社交分享做准备。

## 预备知识

本教程假设你对 React 和 JavaScript 有一个基本的了解。如果你不了解 React，我们推荐你先学习 [React 基础](./react_foundations.md)，了解组件、属性、状态、钩子，以及一些新功能，比如服务器组件和 Suspense 组件。

## 系统要求

在开始本教程之前，你需要确保你的系统满足以下要求：

- 安装了 Node.js 18.17.0 或更新版本。[在这里下载](https://nodejs.org/en)。
- 操作系统：macOS，Windows（包括 WSL），或者 Linux。

另外，你还需要有一个 [Github 账户](https://github.com/join/)和 [Vercel 账户](https://vercel.com/signup)。

# 第一章 开始准备

## 创建一个新项目

打开终端，[cd](https://developer.mozilla.org/en-US/docs/Learn/Tools_and_testing/Understanding_client-side_tools/Command_line#basic_built-in_terminal_commands) 到你想要保存项目的文件夹，然后运行如下命令，来创建一个 Next.js 应用：

```shell
npx create-next-app@latest nextjs-dashboard --use-npm --example "https://github.com/vercel/next-learn/tree/main/dashboard/starter-example"
```

该命令使用了 [`create-next-app`](https://nextjs.org/docs/app/api-reference/create-next-app)，这是一个命令行界面（CLI）工具，可以帮助你搭建一个 Next.js 应用。在上述命令中，我们也使用了 `--example` 参数，意思是我们将在本教程中使用这个[初学者案例](https://github.com/vercel/next-learn/tree/main/dashboard/starter-example)。

## 浏览项目

与其他要从零写代码的教程不同，本教程的大部分代码我们都为你准备好了。这也跟实际开发时差不多，大部分时候你都是接手一个已经存在的项目并在上面工作。

我们的目标是帮助你专注学习 Next.js 的主要功能，而不是编写一个应用的 *全部* 代码。

安装之后，在代码编辑器中打开项目，然后打开 `nextjs-dashboard`。

```shell
cd nextjs-dashboard
```

让我们花些时间看看这个项目。

## 文件夹结构

你会发现这个项目有着如下的文件夹结构：

![learn-folder-structure](learn_nextjs_images/learn-folder-structure.avif)

- **`/app`**：包含了所有的路由，组件，和应用的逻辑代码，大部分时候你都在这里工作。
- **`/app/lib`**：包含应用使用的函数，比如一些可复用的工具函数或者数据抓取函数。
- **`/app/ui`**：包含了应用的 UI 组件，比如页面卡、表格、表单等。为了节省时间，我们已经为你编写好了这些组件的样式代码。
- **`/public`**：包含了应用的所有静态资产，比如图片。
- **`/scripts`**：包含了一个填充脚本，用来在之后的章节中填充数据库。
- **配置文件**：在应用的根目录下还会有一些配置文件，比如 `next.config.js`。大部分文件都在你使用 `create-next-app` 创建一个新项目的时候为你创建并配置好了。本教程中你不需要修改这些文件。

你可以尽管浏览文件夹里的所有内容，如果有不懂的，也不用担心。

## 占位数据

当你构建用户界面的时候，有一些占位数据总是好的。如果还没有数据库或者 API 的话，你可以：

- 使用 JSON 格式的数据或者 JavaScript 对象作占位
- 使用第三方服务比如说 [mockAPI](https://mockapi.io/)。

对于这个项目，我们在 `app/lib/placeholder-data.js` 中提供了一些占位数据。每一个 JavaScript 对象都表示数据库里的一个表。比如说，对于单据表：

```js
const invoices = [
  {
    customer_id: customers[0].id,
    amount: 15795,
    status: 'pending',
    date: '2022-12-06',
  },
  {
    customer_id: customers[1].id,
    amount: 20348,
    status: 'pending',
    date: '2022-11-14',
  },
  // ...
];
```

在[设置数据库](#第六章-设置数据库)的章节中，我们会使用这些数据来填充数据库（初始化数据库）。

## TypeScript

你可能也注意到了很多文件都有 `.ts` 或者 `.tsx` 后缀。这是因为这个项目是用 TypeScript 编写的，我们想要编写一个符合现代网站景观的教程。

如果你不懂 TypeScript 也没关系，我们会在需要的时候提供 TypeScript 代码片段。

现在，让我们看一下 `/app/lib/definitions.ts` 这个文件。这里，我们手动定义了一些会从数据库返回的类型，比如说，单据表会返回如下类型：

```ts
export type Invoice = {
  id: string;
  customer_id: string;
  amount: number;
  date: string;
  // In TypeScript, this is called a string union type.
  // It means that the "status" property can only be one of the two strings: 'pending' or 'paid'.
  status: 'pending' | 'paid';
};
```

使用 TypeScript 可以避免你不小心把错误的数据类型传递给你的组件或者数据库，比如说把一个 `string` 当作 `number` 传递给了 `amount`。

> **如果你是一个 TypeScript 开发者：**
> - 这里我们手动声明了数据类型，但是为了更好的类型安全，我们推荐用 [Prisma](https://www.prisma.io/)，它可以根据你的数据表结构自动生成类型。
> - Next.js 会检测你的项目是否使用了 TypeScript，如果使用了会自动安装必要的包和设置。Next.js 也有一个 [TypeScript 插件](https://nextjs.org/docs/app/building-your-application/configuring/typescript#typescript-plugin)，可以安装在代码编辑器上帮助你自动补全和保证类型安全。

## 运行开发服务器

运行 `npm i` 安装项目的包

```shell
npm i
```

然后运行 `npm run dev` 打开开发服务器

```shell
npm run dev
```

`npm run dev` 会在 `3000` 端口上打开一个 Next.js 开发服务器。让我们来看看这是否管用。在浏览器上打开 http://localhost:3000，你的主页应该像下面这样：

![acme-unstyled](learn_nextjs_images/acme-unstyled.avif)





# 第六章 设置数据库
