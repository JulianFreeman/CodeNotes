# 介绍

## React 基础

为了更有效地学习 Next.js，我们需要对 JavaScript、React，和一些相关的前端开发概念有所了解。但是 JavaScript 和 React 都是很大的话题，我们怎么知道什么时候就可以开始学习 Next.js 了呢？

欢迎来到 React 基础课程！这是一个对初学者友好，由案例引导的课程，它会帮助你了解学习 Next.js 的必备知识。你会一步步构建一个简单的项目，先从一个 JavaScript 应用开始，然后一点点整合到 React 和 Next.js
。

每一章节都是基于前一章节构建的，所以你可以根据自己知道多少来选择从哪里开始。

Next.js 是一个灵活的 **React 框架**，提供了许多构建模块来创建快速的全栈 **网页应用**。

但这具体是什么意思？让我们先花点时间看一下 React 和 Next.js 到底是什么，以及它们能做什么。

## 一个网页应用的构建模块

如果你要构建一个现代的网页应用，就需要考虑很多方面的内容，比如：

- **用户界面**：用户如何与你的应用交互。
- **路由**：用户如何在应用的不同部分进行切换。
- **数据抓取**：你的数据存在哪里以及如何获取。
- **渲染**：何时何地该渲染静态内容或者动态内容。
- **整合**：你使用什么第三方应用（CMS，用户验证，支付功能等）以及如何连接到它们。
- **基础设施**：你在哪里部署、存储，和运行你的应用代码（无服务器、CND、Edge 等）。
- **性能**：如何为用户优化应用性能。
- **可扩展性**：当你的团队、数据、用户流量逐渐增长时你的应用如何适应。
- **开发体验**：你团队开发和维护该应用的体验。

对于你应用的每一部分，你都需要思考你是自己想办法解决，还是使用第三方的库或者框架解决。

## React 是什么？

[React](https://react.dev/learn) 是一个用于构建交互式 **用户界面** 的 JavaScript **库**。

这里的用户界面，指的是用户在屏幕上看到的并且可以交互的各种元素。

![user-interface](images/user-interface.avif)

这里的库，指的是 React 提供了很多很有用的功能来构建 UI，但是由开发者自己决定在哪里使用这些功能。

React 能够成功的一部分原因也在于它相对不那么关心构建应用的其他方面。这也造就了一个满是第三方工具和解决方案的丰富的生态系统。

这也意味着，要从头开始构建一个完整的 React 应用，是需要花些工夫的。开发者需要花时间配置各种工具以及为某些常见的应用需求重复造轮子。

## Next.js 是什么？

Next.js 是一个 React **框架**，提供了创建网页应用的构建模块。

这里的框架，指的是 Next.js 负责处理 React 需要的工具配置，也提供了一些额外的工具，功能和优化。

![next-app](images/next-app.avif)

你可以使用 React 来构建你的 UI，然后逐渐用 Next.js 的功能来解决常见的应用需求，比如路由、数据抓取、整合等，一步步提升开发体验和用户体验。

不管你是独立开发者，还是一个大团队的一员，你都可以使用 React 和 Next.js 来构建一个全交互式、高动态的、优秀的网页应用。

在接下来的几个章节，我们会讨论如何使用 React 和 Next.js。

## 预备知识

本教程需要一些 HTML，CSS，JavaScript 基础，但不需要了解 React。如果你已经了解 React 了，你可以直接跳到[开始使用 Next.js](https://nextjs.org/learn/foundations/from-react-to-nextjs/getting-started-with-nextjs) 或者[创建你的第一个 Next.js 应用](https://nextjs.org/learn/basics/create-nextjs-app)。

# 第一章 渲染用户界面

要理解 React 是如何工作的，我们需要对浏览器是如何解析我们的代码并创建一个交互式用户界面的。

当用户访问一个网页时，服务器会返回一个 HTML 文件给浏览器，这个文件可能像如下的样子：

![html-to-dom](images/html-to-dom.avif)

然后浏览器读取这个 HTML 文件并构建了一个文件对象模型（DOM）。

## 什么是 DOM？

DOM 是一种对 HTML 元素的呈现，这种呈现是基于对象的。它是代码和用户界面之间的桥梁，有树状结构并包含一些父子关系。

![dom-to-ui](images/dom-to-ui.avif)

你可以使用 DOM 方法和一门编程语言，比如 JavaScript，来监听用户事件，[修改 DOM 结构](https://developer.mozilla.org/docs/Learn/JavaScript/Client-side_web_APIs/Manipulating_documents)（选择、添加、更新、删除用户界面上的特定元素）。修改 DOM 不仅是针对特定的元素，也可以修改元素的样式和内容。

我们在下一节学习如何使用 JavaScript 和 DOM 方法。

# 第二章 用 JavaScript 更新 UI

让我们看看该如何使用 JavaScript 和 DOM 来给我们的项目里插入一个 `h1` 标签。

打开你的代码编辑器，然后创建一个新的 `index.html` 文件，在文件中添加如下内容：

```html
<html>
  <body>
    <div></div>
  </body>
</html>
```

然后给 `div` 一个唯一的 `id` 以便后续我们可以选中它。

```html
<html>
  <body>
    <div id="app"></div>
  </body>
</html>
```

要想在 HTML 中编写 JavaScript 代码，添加一个 `script` 标签。

```html
<html>
  <body>
    <div id="app"></div>
    <script type="text/javascript"></script>
  </body>
</html>
```

现在，在 `script` 标签内，你可以使用 DOM 方法 `getElementById()`，然后通过 `id` 选中 `<div>` 元素。

```html
<html>
  <body>
    <div id="app"></div>
 
    <script type="text/javascript">
      const app = document.getElementById('app');
    </script>
  </body>
</html>
```

然后你可以继续使用 DOM 方法创新一个新的 `h1` 元素。

```html
<html>
  <body>
    <div id="app"></div>
 
    <script type="text/javascript">
      // Select the div element with 'app' id
      const app = document.getElementById('app');
 
      // Create a new H1 element
      const header = document.createElement('h1');
 
      // Create a new text node for the H1 element
      const text = 'Develop. Preview. Ship. 🚀';
      const headerContent = document.createTextNode(text);
 
      // Append the text to the H1 element
      header.appendChild(headerContent);
 
      // Place the H1 element inside the div
      app.appendChild(header);
    </script>
  </body>
</html>
```

为了确保一切正常，在浏览器里打开这个 HTML 文件，你应该会看到一个 h1 标签，写着“Develop. Preview. Ship. 🚀”。

## HTML 和 DOM

如果你在[浏览器开发工具](https://developer.chrome.com/docs/devtools/overview/)中查看 DOM 元素，你会发现 DOM 中包含一个 `<h1>` 元素。页面的 DOM 结构和我们的源码不一致。

![source-code](images/source-code.avif)

这是因为 HTML 呈现的是 **原始的页面内容**，而 DOM 呈现的是 **更新后的页面内容**，我们刚刚用 JavaScript 更新的。

使用纯 JavaScript 代码更新 DOM，很厉害，但也很繁琐。你需要写这么多代码才能把一个 `<h1>` 元素插入到页面中：

```html
<script type="text/javascript">
  const app = document.getElementById('app');
  const header = document.createElement('h1');
  const text = 'Develop. Preview. Ship. 🚀';
  const headerContent = document.createTextNode(text);
  header.appendChild(headerContent);
  app.appendChild(header);
</script>
```

当应用或者团队越来越大的时候，用这种方式构建应用会变得越来越棘手。

这种方式需要开发者花很多时间写很多代码告诉计算机它要 **如何** 工作。但是如果我们只需要描述我们想要 **什么**，然后让计算机自己处理该 **如何** 更新 DOM，岂不美哉？

## 命令式编程和声明式编程

上面的代码是一个 **命令式编程** 的很好的例子。你一步步写代码告诉计算机该 **如何** 更新用户界面。但是对于构建用户界面，往往声明式才是更好的方法，因为它可以加快开发进度。相较于使用 DOM 方法，如果开发者可以直接声明他们想展示 **什么**（此例中即一个 `h1` 标签以及一些文字）会更好。

换句话说，**命令式编程** 就像你告诉一个厨师如何一步步制作一个披萨。**声明式编程** 则是直接点一个披萨，而不需要关心这个披萨是怎么一步步做出来的。🍕

[React](https://react.dev/) 就是一个知名的帮助开发者构建用户界面的声明式编程库。

## React：一个声明式 UI 库

作为一个开发者，你可以告诉 React 你想要什么样的用户界面，然后 React 会自己想办法更新 DOM。

下一节，我们会开始讨论如何使用 React。

## 第三章 开始使用 React


