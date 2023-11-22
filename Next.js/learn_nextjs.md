# 学习 Next.js

欢迎来到 Next.js 应用路由教程！在这个免费的交互式课程中，你将会通过构建一个全栈网页应用来学习 Next.js 的主要功能。

> 译注：当然本翻译文件不可能为交互式的啦~

## 我们要构建什么

![dashboard](./learn_nextjs_images/dashboard.avif)

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
- **路由**：如何使用文件系统路由创建嵌套的布局和页面。
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

```sh
npx create-next-app@latest nextjs-dashboard --use-npm --example "https://github.com/vercel/next-learn/tree/main/dashboard/starter-example"
```

该命令使用了 [`create-next-app`](https://nextjs.org/docs/app/api-reference/create-next-app)，这是一个命令行界面（CLI）工具，可以帮助你搭建一个 Next.js 应用。在上述命令中，我们也使用了 `--example` 参数，意思是我们将在本教程中使用这个[初学者案例](https://github.com/vercel/next-learn/tree/main/dashboard/starter-example)。

## 浏览项目

与其他要从零写代码的教程不同，本教程的大部分代码我们都为你准备好了。这也跟实际开发时差不多，大部分时候你都是接手一个已经存在的项目并在上面工作。

我们的目标是帮助你专注学习 Next.js 的主要功能，而不是编写一个应用的 *全部* 代码。

安装之后，在代码编辑器中打开项目，然后打开 `nextjs-dashboard`。

```sh
cd nextjs-dashboard
```

让我们花些时间看看这个项目。

## 文件夹结构

你会发现这个项目有着如下的文件夹结构：

![learn-folder-structure](./learn_nextjs_images/learn-folder-structure.avif)

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

```sh
npm i
```

然后运行 `npm run dev` 打开开发服务器

```sh
npm run dev
```

`npm run dev` 会在 `3000` 端口上打开一个 Next.js 开发服务器。让我们来看看这是否管用。在浏览器上打开 http://localhost:3000 ，你的主页应该像下面这样：

![acme-unstyled](./learn_nextjs_images/acme-unstyled.avif)

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

![home-page-with-tailwind](./learn_nextjs_images/home-page-with-tailwind.avif)

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

在之前的章节中，我们学习了如何给 Next.js 应用设置样式。现在让我们继续调整主页，添加一个自定义字体和一张主页图。

> **本章我们会讲到……**
> - 如何使用 `next/font` 添加自定义字体。
> - 如何使用 `next/image` 添加图片。
> - Next.js 是如何优化字体和图片的。

## 为什么要优化字体？

字体在网站设计中占据了相当重要的位置，但如果使用自定义字体时字体文件需要临时抓取和加载，那么就会很影响项目的性能。

[累计布局偏移](https://web.dev/cls/)（CLS）是一项谷歌用来评估一个网站性能和用户体验的指标。当一个网站先用备用字体或者系统字体进行初始化渲染，等自定义字体加载完成后再将字体替换，这样就发生了布局偏移。这种替换会使得字体大小、间隔或者整个布局发生变化，也使得其周围的元素发生偏移。

![font-layout-shift](./learn_nextjs_images/font-layout-shift.avif)

当你使用 `next/font` 模块的时候，Next.js 会自动为你的应用优化字体。它会在构建时期把字体文件下载下来，然后把它们跟你的其他静态文件放在一起管理。这意味着当有人访问你的网站时，网站不需要额外的网络请求来下载字体，这就提升了网站性能。

## 添加主字体

让我们为你的应用添加一个自定义的谷歌字体吧！

在 `/app/ui` 文件夹，创建一个名为 `fonts.ts` 的新文件。我们将使用这个文件来保存应用中所使用的字体。

从 `next/font/google` 模块导入 `Inter` 字体，这将会是我们的主字体。然后指定要加载哪一个[子集](https://fonts.google.com/knowledge/glossary/subsetting)，在我们的例子中，是 `"latin"`。

```ts
import { Inter } from 'next/font/google';
 
export const inter = Inter({ subsets: ['latin'] });
```

最后，在 `/app/layout.tsx` 文件的 `<body>` 元素中添加该字体。

```tsx
import '@/app/ui/global.css';
import { inter } from '@/app/ui/fonts';
 
export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body className={`${inter.className} antialiased`}>{children}</body>
    </html>
  );
}
```

给 `<body>` 元素添加 `Inter` 之后，这个字体就会应用到整个应用中。这里，我们也添加了一个 Tailwind 类名 `antialiased` 来使字体更平滑。这不是必需的，但是加上会很不错。

打开浏览器，打开开发者工具，选择 `body` 元素，你应该会看到 `Inter` 和 `Inter_Fallback` 出现在样式中。

## 练习：添加第二字体

你可以给应用中特定的元素添加字体。

现在你自己来操作吧！在 `fonts.ts` 文件中，导入名为 `Lusitana` 的第二字体，然后把它传递给 `/app/page.tsx` 文件的 `<p>` 元素。要像前面那样指定字体子集的话，你还需要指定字体的 **字重**。

当你完成了，可以展开下面的代码片段查看解决方案。

> **提示**：
> - 如果你不知道怎么传递字体的字重选项，看一下代码编辑器报的 TypeScript 错误。
> - 访问[谷歌字体](https://fonts.google.com/)并搜索 `Lusitana` 来查看有哪些子集可选。
> - 查看文档了解如何[添加多个字体](https://nextjs.org/docs/app/building-your-application/optimizing/fonts#using-multiple-fonts)和[所有可用选项](https://nextjs.org/docs/app/api-reference/components/font#font-function-arguments)。

> 解决方案如下（当然无法折叠啦~）

`/app/ui/fonts.ts`

```ts
import { Inter, Lusitana } from 'next/font/google';
 
export const inter = Inter({ subsets: ['latin'] });
 
export const lusitana = Lusitana({
  weight: ['400', '700'],
  subsets: ['latin'],
});
```

`/app/page.tsx`

```tsx
import AcmeLogo from '@/app/ui/acme-logo';
import { ArrowRightIcon } from '@heroicons/react/24/outline';
import Link from 'next/link';
import { lusitana } from '@/app/ui/fonts';
 
export default function Page() {
  return (
    // ...
    <p
      className={`${lusitana.className} text-xl text-gray-800 md:text-3xl md:leading-normal`}
    >
      <strong>Welcome to Acme.</strong> This is the example for the{' '}
      <a href="https://nextjs.org/learn/" className="text-blue-500">
        Next.js Learn Course
      </a>
      , brought to you by Vercel.
    </p>
    // ...
  );
}
```

最后 `<AcmeLogo />` 组件也使用了 Lusitana 字体，之前被注释了，为了防止报错，现在你可以移除它的注释了。

```tsx
// ...
 
export default function Page() {
  return (
    <main className="flex min-h-screen flex-col p-6">
      <div className="flex h-20 shrink-0 items-end rounded-lg bg-blue-500 p-4 md:h-52">
        <AcmeLogo />
        {/* ... */}
      </div>
    </main>
  );
}
```

很好，你现在已经给应用添加了两个自定义字体了，下面，让我们给主页添加一张主页图。

## 为什么要优化图片？

Next.js 在顶层文件夹 `/public` 中管理静态资源，比如图片。`/public` 文件夹内的文件都可以被应用使用。

用常规的 HTML 语法，你需要像这样添加图片：

```html
<img
  src="/hero.png"
  alt="Screenshots of the dashboard project showing desktop version"
/>
```

然而，这也意味着你需要手动做这些：

- 确保你的图片可以在不同的屏幕大小上做出合适的响应
- 为不同的设备指定图片大小
- 防止图片加载时造成的布局偏移
- 在用户视窗之外时延迟加载图片

图片优化是网站开发中的一个大话题，甚至可以自成一派。为了避免手动实现这些优化细节，你可以使用 `next/image` 组件来自动优化图片。

## `<Image>` 组件

`<Image>` 组件是对 HTML `<img>` 标签的扩展，具有自动图片优化的功能，包括：

- 在图片加载时防止自动布局偏移
- 可以自动调整大小，以避免在小窗口的设备上加载大图片
- 默认延迟加载图片（当进入用户视窗时加载图片）
- 如果浏览器支持的话，用现代的图片格式，如 [WebP](https://developer.mozilla.org/en-US/docs/Web/Media/Formats/Image_types#webp) 和 [AVIF](https://developer.mozilla.org/en-US/docs/Web/Media/Formats/Image_types#avif_image) 管理图片

## 添加桌面端主页图

让我们来使用 `<Image>` 组件吧。如果你打开 `/public` 文件夹，你会看到有两张图片：`hero-desktop.png` 和 `hero-mobile.png`。两张图完全不同，它们一张用于显示在桌面端设备，另一张用于移动端。

在 `/app/page.psx` 文件中，从 [`next/image`](https://nextjs.org/docs/api-reference/next/image) 中导入组件，然后在注释下面添加图片：

```tsx
import AcmeLogo from '@/app/ui/acme-logo';
import { ArrowRightIcon } from '@heroicons/react/24/outline';
import Link from 'next/link';
import { lusitana } from '@/app/ui/fonts';
import Image from 'next/image';
 
export default function Page() {
  return (
    // ...
    <div className="flex items-center justify-center p-6 md:w-3/5 md:px-28 md:py-12">
      {/* Add Hero Images Here */}
      <Image
        src="/hero-desktop.png"
        width={1000}
        height={760}
        className="hidden md:block"
        alt="Screenshots of the dashboard project showing desktop version"
      />
    </div>
    //...
  );
}
```

这里，我们把 `width` 设置为 `1000` 像素，把 `height` 设置为 `760` 像素。手动设置图片的宽高来避免布局偏移是一个好办法，这个宽高比需要跟原图片的宽高比 **相同**。

这里我们还添加了 `hidden` 类来从移动端屏幕中移除图片，和 `md:block` 来在桌面端屏幕中显示图片。

![home-page-with-hero](./learn_nextjs_images/home-page-with-hero.avif)

## 练习：添加移动端的主页图

现在到你了！在刚才添加的图片下面，添加一张 `hero-mobile.png` 的 `<Image>` 组件。

- 图片的 `width` 应该为 `560` 像素，`height` 应该为 `620` 像素。
- 图片应该在移动端屏幕上显示，在桌面端隐藏。你可以通过开发者工具查看图片在桌面端和移动端中是否正常切换。

当你完成之后，可以展开下面的代码查看解决方案。

> 解决方案如下（当然无法折叠啦~）

`/app/page.tsx`

```tsx
import AcmeLogo from '@/app/ui/acme-logo';
import { ArrowRightIcon } from '@heroicons/react/24/outline';
import Link from 'next/link';
import { lusitana } from '@/app/ui/fonts';
import Image from 'next/image';
 
export default function Page() {
  return (
    // ...
    <div className="flex items-center justify-center p-6 md:w-3/5 md:px-28 md:py-12">
      {/* Add Hero Images Here */}
      <Image
        src="/hero-desktop.png"
        width={1000}
        height={760}
        className="hidden md:block"
        alt="Screenshots of the dashboard project showing desktop version"
      />
      <Image
        src="/hero-mobile.png"
        width={560}
        height={620}
        className="block md:hidden"
        alt="Screenshot of the dashboard project showing mobile version"
      />
    </div>
    //...
  );
}
```

很好，现在我们的主页有自定义字体和主页图了。

## 推荐阅读

关于这些话题，还有很多内容，比如优化远程图片，或者使用本地的字体文件等。如果想了解更多内容，可以阅读下面的文档：

- [图片优化文档](https://nextjs.org/docs/app/building-your-application/optimizing/images)
- [字体优化文档](https://nextjs.org/docs/app/building-your-application/optimizing/fonts)
- [从图片角度提升网站性能（MDN）](https://developer.mozilla.org/en-US/docs/Learn/Performance/Multimedia)
- [网站字体（MDN）](https://developer.mozilla.org/en-US/docs/Learn/CSS/Styling_text/Web_fonts)

# 第四章 创建布局和页面


# 第六章 设置数据库
