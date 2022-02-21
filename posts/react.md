form validation and sanitization

One important part of using forms to record and store user input 
is letting 
users know 
when they’re violating any validation rules you’ve set up 
and when they’re doing anything to provide data that doesn’t satisfy your application.

violating 违反
validation 校验，验证
satisfy 满足

Hopefully, 
the server applications that receive data from your client-side application 
will have strict data-validation and sanitization procedures in place—you can’t rely on a browser app to do all the work in this area.

procedure 程序，步骤，手续

And even if you have good data sanitization and validation in place on your server,
you still need to provide and enforce good data practices on the front end in order to help users, 
to add another level of defense against bad actors, 
and to promote data integrity. 

enforce 执行
defense 防御
promote 提升
integrity 完整性

If you don’t, you could potentially have confused users, security holes, and meaningless data—all things you don’t want.

potentially 潜在的
confused 困惑
security holes 安全漏洞

As we’ve seen so far, 
using forms to update component state 
involves 
state, props, and component methods, 
like much else in React.

As we’ve seen so far 正如我们到目前为止所看到的，正如目前所看到的
involves 包含，涉及
like much else 像其他很多东西一样

To add validation and sanitization to your component, you need to hook into the update process.

To that end, 
you’ll be writing general-purpose validation and sanitization functions 
that could be used anywhere 
you can use JavaScript 
and probably in most other front-end frameworks.

To that end 为此，为了达到这个目的

Fortunately, the CreatePost component you’re creating doesn’t require extensive validation. 

extensive 大量

You only need to check for a maximum length 
and do some additional validation 
so the component won’t submit empty posts to the API server.

so 以便

We’re using a simple server setup for the purposes of learning and local development, 
so it will accept most payloads without doing much validation.

Writing applications on the server is another domain outside the scope of this book, so I’ll only focus on validation and sanitization on the browser.

You need to ask yourself a few questions when setting up validation for forms and inputs in your applications:

What are the data requirements for the application?
Based on these constraints, how can you help your users provide meaningful data?
Are there ways you can eliminate inconsistencies in data that users provide?

eliminate 消除
inconsistencies 矛盾，不一致，（一致性）

First, you need to find out 
what the data requirements 
set by the business or application back end (if one exists) 
are. 

find out 找出，明确，了解

You should start there 
because that knowledge will help you 
establish 
basic guidelines 
for how to treat your data. 

treat 对待，处理
basic guidelines 基本准则，基本的指导方针

Because we’ve already established 
that your server will willingly accept most things 
and we’ve set out the basic data types for a post, 
we can move on the next question.

Based on the constraints you have, 
how can you best help your users provide meaningful data and have a good experience in your app?

constraints 约束，约束条件
experience 体验

That usually involves checking data for things like size, character type, maybe file type for file uploads, and more. 

Right now, your CreatePost component is fairly benign, and there’s not much to validate beyond length. 

fairly 相当
benign 良性的，不错

Next you’ll check for a minimum and maximum length and only let the user submit their post if valid. 

The following listing shows how to set up some basic validation for your component.

Create a simple valid property in local component state

Determine validity of post by setting max length here—280 demonstrates usage, but users sometimes want posts to be long

Determine 确定
setting 设定
demonstrates 演示
usage 用法

Create a new post object

We’ve worked on answering the first two questions (data constraints and validation).

Now we can approach the final aspect: eliminating data inconsistencies with (very) basic data sanitization. 

approach 接近
aspect 方面
sanitization 搞卫生，清理

Whereas validation is asking the user for certain data, 
sanitization is ensuring that the data you get back is safe and in the right format, 
and that it exists in a way that it can be persisted.

ensuring 确保
persisted 存留

Information security is a huge and very important field, 
and this book can’t begin to really go into proper data handling for security
—but we can tackle one smaller area for Letters: 
offensive content.

proper 适当的
tackle 应付
offensive 攻击

You’ll use a JavaScript module called bad-words, 
available from npm 
(the main JavaScript module registry and service—learn more at www.npmjs.com/about), 
to help us out. 

called 名为，称为

It should already be installed in your project. 

bad-words takes in a string and replaces any words found on a blacklist 
(you can create your own and substitute it for the default if you prefer) 
with asterisks.

substitute 代替
asterisks 星号

The example illustrated in the following listing is mostly contrived, 
but at the very least you can prevent people from posting potentially offensive content on the public app (https://social.react.sh)

mostly 主要的
contrive 设计，发明，人为的
prevent 防止
potentially 潜在的
offensive 讨厌的，攻击的

Remember, this a very contrived example and isn’t in any way suggesting or endorsing any kind of censorship.

endorsing 认同
censorship 审查

Import default object from bad-words module

Use a constructor to create new instance of filter

Pass form value into the .clean() method of filter and use returned value to set state

Pass 传递，通过，经过



