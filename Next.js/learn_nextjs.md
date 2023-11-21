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

`npm run dev` 会在 `3000` 端口上打开一个 Next.js 开发服务器。让我们来看看这是否管用。在浏览器上打开 http://localhost:3000 ，你的主页应该像下面这样：

![acme-unstyled](learn_nextjs_images/acme-unstyled.avif)

# 第二章 CSS 样式

目前为止，你的主页上还没有任何样式。让我们看看下面几种给 Next.js 应用设置样式的不同方式。

> **本章我们会讲到……**
> - 怎么为应用添加全局 CSS 文件。
> - 两种不同的设置样式的方式：Tailwind 和 CSS modules。
> - 如何使用 `clsx` 工具包有选择地添加类名。

## 全局样式

如果你观察 `/app/ui` 文件夹，你会发现里面有一个文件叫 `global.css`。你可以通过在这个文件中添加 CSS 规则来给应用的 **所有** 页面添加样式，比如 CSS 重置规则，HTML 元素（比如链接）的全站样式等。

你可以在应用的任意组件内导入 `global.css`，不过通常来说最好把它加入到最顶级的组件中。在 Next.js 中，这个顶级组件是 [root layout](https://nextjs.org/docs/app/building-your-application/routing/pages-and-layouts#root-layout-required)（稍后讨论）。

找到 `/app/layout.tsx` 文件，然后导入 `global.css` 文件，以为你的应用添加全局样式。

```tsx
import '@/app/ui/global.css';
 
export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}
```

确保开发服务器正常运行，保存修改，然后在浏览器里查看，你的主页现在应该如下图所示：

![home-page-with-tailwind](learn_nextjs_images/home-page-with-tailwind.avif)

但是等等，你并没有添加任何 CSS 规则，这些样式是哪来的？

如果你看一看 `global.css` 的内容的话，你会发现有几个 `@tailwind` 声明：

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

## Tailwind

[Tailwind](https://tailwindcss.com/) 是一个 CSS 框架，通过允许我们直接在 TSX 标记语言中编写[辅助类](https://tailwindcss.com/docs/utility-first)来帮助我们提高开发进度。

使用 Tailwind 时，你通过添加类名来给元素添加样式，比如说，给 `<h1>` 添加 `"text-blue-500"` 会把文本变蓝：

```html
<h1 className="text-blue-500">I'm blue!</h1>
```

尽管 CSS 样式是全局共享的，但是每个类只单独应用于每个元素。这意味着如果你添加或者删除一个元素，你不用担心去维护额外的样式表、样式冲突，或者担心随着应用越来越大你的 CSS 包也越来越大。

当你使用 `create-next-app` 创建新的项目时，Next.js 会询问你是否想要用 Tailwind。如果你选择 `yes`，Next.js 会自动安装必要的包并为你的应用配置 Tailwind。

如果你看一下 `/app/page.tsx` 文件，你会发现我们这个例子中正在使用 Tailwind 的类。

```tsx
import AcmeLogo from '@/app/ui/acme-logo';
import { ArrowRightIcon } from '@heroicons/react/24/outline';
import Link from 'next/link';
 
export default function Page() {
  return (
    // These are Tailwind classes:
    <main className="flex min-h-screen flex-col p-6">
      <div className="flex h-20 shrink-0 items-end rounded-lg bg-blue-500 p-4 md:h-52">
    // ...
  )
}
```

如果这是你第一次使用 Tailwind，不必担心。为了节省时间，我们已经为你把所有要使用的组件都设置好样式了。

让我们玩玩 Tailwind 吧！把下面的代码拷贝并粘贴在 `/app/page.tsx` 文件的 `<p>` 元素的上面。

```html
<div
  className="h-0 w-0 border-b-[30px] border-l-[20px] border-r-[20px] border-b-black border-l-transparent border-r-transparent"
/>
```

如果你更喜欢传统的 CSS 规则，或者你想把你的样式跟 JSX 隔离开，那么 CSS Modules 会是一个不错的选择。

## CSS Modules

[CSS Modules](https://nextjs.org/docs/basic-features/built-in-css-support) 可以在你把 CSS 写入到组件中时自动创建唯一的类名，这样你也不用担心样式冲突。

在本教程我们会使用 Tailwind，但是也让我们来看一看如何使用 CSS Modules 实现上例中相同的效果吧。

在 `/app/ui` 中，创建一个名为 `home.module.css` 的新文件，然后添加如下规则：

```css
.shape {
  height: 0;
  width: 0;
  border-bottom: 30px solid black;
  border-left: 20px solid transparent;
  border-right: 20px solid transparent;
}
```

然后在 `/app/page.tsx` 文件中导入样式，并把你之前添加的 `<div>` 中的 Tailwind 类名替换为 `styles.shape`。

```tsx
import styles from '@/app/ui/home.module.css';
 
//...
<div className="flex flex-col justify-center gap-6 rounded-lg bg-gray-50 px-6 py-10 md:w-2/5 md:px-20">
    <div className={styles.shape}></div>;
// ...
```

保存更改并在浏览器中查看效果。你应该会看到一个跟之前一样的图形。

Tailwind 和 CSS Modules 是给 Next.js 应用添加样式的两大常用方式。用哪一个纯粹是个人喜好了，你甚至也可以都用。

## 使用 `clsx` 库来切换类名

可能有些时候，你需要根据一些状态或者其他情况更改元素的样式。

[`clsx`](https://www.npmjs.com/package/clsx) 是一个可以让你轻松切换类名的库。我们建议查看[官方文档](https://github.com/lukeed/clsx)了解更多细节，下面是一些基本使用：

- 假设你想创建一个名为 `InvoiceStatus` 的组件并接收参数 `status`，这个参数可能是 `"pending"` 或者 `"paid"`。
- 如果是 `"paid"`，颜色就变为绿色，如果是 `"pending"`，颜色就变为灰色。

你可以使用 `clsx` 来根据条件应用不同的类：

```tsx
import clsx from 'clsx';
 
export default function InvoiceStatus({ status }: { status: string }) {
  return (
    <span
      className={clsx(
        'inline-flex items-center rounded-full px-2 py-1 text-sm',
        {
          'bg-gray-100 text-gray-500': status === 'pending',
          'bg-green-500 text-white': status === 'paid',
        },
      )}
    >
    // ...
)}
```

## 其他设置样式的方案

除了我们前面提到的方法，你也可以通过如下方法给 Next.js 应用添加样式：

- Sass，它允许你导入 `.css` 和 `.scss` 文件。
- CSS-in-JS 库，比如 [styled-jsx](https://github.com/vercel/styled-jsx)，[styled-components](https://github.com/vercel/next.js/tree/canary/examples/with-styled-components)，和 [emotion](https://github.com/vercel/next.js/tree/canary/examples/with-emotion)。

查看 [CSS 文档](https://nextjs.org/docs/app/building-your-application/styling)了解更多信息。

# 第三章 优化字体和图片



# 第六章 设置数据库
