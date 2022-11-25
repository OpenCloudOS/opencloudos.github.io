# Documentation format guide

This page lists the some rules that should be followed when writing OpenCloudOS documentation. Please read this page carefully before writing or modifying documentation to help you write higher quality content.

## Storage format and document links

1. **File names are written in lowercase letters, and words are separated by `-`.** For example: `docs-format-guide.md`.
2. **Add `.en` after the file name of an English documentation.** For example: `docs-format-guide.en.md`. Files for English documentations are stored in the same directory as Chinese documentations.
3. Use relative paths for internal links, for example: `[Format Guide](docs-format-guide.md)`, `[FAQ](../faq.md)`.
4. All the images used are stored in the `docs/assets` directory of the repository and have a meaningful file names. Use relative paths for image references, for example: `![OpenCloudOS favicon](./assets/favicon.png)`.

## Basic format requirements for documents

- Level 1 headings in form of `#` or `<h1>` are not allowed in the document body except for the title of the document, which is the same as the title of the document and written in the first line of the document.
- A space is required after the number sign of headings. For example: `## Second-level heading`.
- A white line is required after a heading.
- Use 4 spaces for indentation, not Tab.
- For block-level elements such as code blocks, tables, etc., there should be a blank line before and after the element.
- An indentation of 4 spaces is required for second-level lists.
- The content added to the same level list should have an indentation of 4 spaces, and two blank lines should be added before and after the content.
- For admonitions using the `???` or `!!!` syntax, an identation of 4 spaces is required for each line of text, even if it is a blank line. An empty line is required before and after the admonitions, but not before and after the content.
- Code blocks used in form of ```` ``` ```` should have a language specified. For example: ```` ``` shell ````. If the code content is plain text, specify `text` as the language.
- Refer to the [Chinese Copywriting Guidelines](#Chinese Copywriting Guidelines) at the end of this document when writing Chinese documentations.

    !!! note
        If necessary, use the tools mentionned in [tools](#tools) section of the Copywriting Guidelines to ensure the good formatting of the text.

## Chinese Copywriting Guidelines

!!! note
    The content below was modified from the "Chinese Copywriting Guidelines" written by GitHub user sparanoid. The original content was licensed under the MIT license. [GitHub repository](https://github.com/sparanoid/chinese-copywriting-guidelines)

### Spacing

> "Research shows that, people adding no space between Chinese and English suffer from pathetic relationships. 70% of them are married by the age of 34, with someone they don't love; 30% of them left everything for their cats and died. Blank spaces are essential to both romance and writing.
>
> Let's do it." ——[vinta/paranoid-auto-spacing](https://github.com/vinta/pangu.js)

#### Place one space before & after English words

Good:

> 目标是打造全面中立、开放、安全、稳定易用、高性能的 Linux 服务器操作系统

Bad:

> 目标是打造全面中立、开放、安全、稳定易用、高性能的Linux服务器操作系统
>
> 目标是打造全面中立、开放、安全、稳定易用、高性能的 Linux服务器操作系统

An example of complete and correct usage:

> OpenCloudOS 是由 20 余家操作系统、云平台、软硬件厂商与个人共同倡议发起的操作系统社区项目，即将进入开放原子开源基金会（OpenAtom Foundation）孵化及运营。目标是打造全面中立、开放、安全、稳定易用、高性能的 Linux 服务器操作系统，共建国产操作系统开源技术社区，扩大社区发行版影响力，构建操作系统健康繁荣的生态。

#### Place one space before & after numbers

Good:

> OpenCloudOS 是由 20 余家操作系统、云平台、软硬件厂商与个人共同倡议发起的操作系统社区项目

Bad:

> OpenCloudOS 是由20余家操作系统、云平台、软硬件厂商与个人共同倡议发起的操作系统社区项目

#### Place one space between numbers and units

Good:

> 使用 4k 扇区驱动器时，最大大小为 16 TiB。

Bad:

> 使用 4k 扇区驱动器时，最大大小为 16TiB。

Exceptions: There should not be any spacing between numbers and degrees/percentages.

Good:

> 角度為 90° 的角，就是直角。
>
> 新 MacBook Pro 有 15% 的 CPU 性能提升。

Bad:

> 角度為 90 ° 的角，就是直角。
>
> 新 MacBook Pro 有 15 % 的 CPU 性能提升。

#### No additional spaces before/after punctuation in fullwidth form

Good:

> chrony 是网络时间协议（NTP）的通用实现。

Bad:

> chrony 是网络时间协议（ NTP ）的通用实现。

#### `text-spacing` to the help?

[`text-spacing`](https://www.w3.org/TR/css-text-4/#text-spacing-property) and [`-ms-text-autospace`](https://msdn.microsoft.com/library/ms531164(v=vs.85).aspx) provided by CSS Text Module Level and Microsoft can specify the autospacing and narrow space width adjustment of text. However it's not popular, and on other platforms such as OS X and iOS we can not use this feature. So it's better for you to keep up the habit.

### Punctuation

### Fullwidth and halfwidth

If you're not familiar with fullwidth and halfwidth forms please refer to article [Halfwidth and fullwidth](https://en.wikipedia.org/wiki/Halfwidth_and_fullwidth_forms) on Wikipedia.

#### Use punctuation in fullwidth form

Good:

> chrony 是网络时间协议（NTP）的通用实现。

Bad:

> chrony 是网络时间协议 (NTP) 的通用实现。
>
> chrony 是网络时间协议(NTP)的通用实现。

#### Use numbers in halfwidth form

Good:

> 使用 4k 扇区驱动器时，最大大小为 16 TiB。

Bad:

> 使用 ４k 扇区驱动器时，最大大小为 １６ TiB。

Exceptions: fullwidth numbers are acceptable for better visual alignment in graphic design.

#### Use punctuation in halfwidth form for English sentences

Good:

> 乔布斯那句话是怎么说的？「Stay hungry, stay foolish.」
>
> 推荐你阅读《Hackers & Painters: Big Ideas from the Computer Age》，非常的有趣。

Bad:

> 乔布斯那句话是怎么说的？「Stay hungry，stay foolish。」
>
> 推荐你阅读《Hackers＆Painters：Big Ideas from the Computer Age》，非常的有趣。

### Nouns

#### Use correct case for proper nouns

The case usage of proper nouns is related to English writing, and is not the topic of this wiki. So only some common mistakes are listed here.

Good:

> 欢迎来到 OpenCloudOS 文档库！
>
> TencentOS Server 是腾讯云针对云的场景研发的 Linux 操作系统。

Bad:

> 欢迎来到 opencloudos 文档库！
>
> 欢迎来到 OPENCLOUDOS 文档库！
>
> 欢迎来到 Opencloudos 文档库！
>
> 欢迎来到 opencloudOS 文档库！
>
> tencentos server 是腾讯云针对云的场景研发的 linux 操作系统。
>
> TENCENTOS SERVER 是腾讯云针对云的场景研发的 LINUX 操作系统。
>
> TencentOS server 是腾讯云针对云的场景研发的 Linux 操作系统。
>
> tencentOS Server 是腾讯云针对云的场景研发的 Linux 操作系统。

Please note that when the text needs to be displayed in all uppercase or all lowercase for visual consistency, please use the standard case in HTML and use `text-transform: uppercase;`/`text-transform: lowercase;` to define the presentation.

#### Avoid jargons

Good:

> 我们需要一位熟悉 TypeScript、HTML5，至少理解一种框架（如 React、Next.js）的前端开发者。

Bad:

> 我们需要一位熟悉 Ts、h5，至少理解一种框架（如 RJS、nextjs）的 FED。

### Dispute

The following usages comprise of personal characteristics. As such, from the perspective of copywriting guidelines, they are **still correct** regardless of whether they comply with the following rules.

#### Add extra spaces before/after links

!!! note
    In this case, we recommend you to treat the link content as normal content, that is, if the content before and after the link is in Chinese and the link name is in Chinese, then you do not need to add spaces before and after the link. Otherwise, add spaces between English/digits and Chinese.

Usage:

> 请 [提交一个 issue](#) 反馈相关问题。
>
> 访问我们网站的最新动态，请 [点击这里](#) 进行订阅！

compared with:

> 请[提交一个 issue](#)反馈相关问题。
>
> 访问我们网站的最新动态，请[点击这里](#)进行订阅！

#### Use corner brackets for Chinese Simplified

!!! note
    We recommend the use of the normal brackets (“” and ‘’).

Usage:

> 「老师，『有条不紊』的『紊』是什么意思？」

compared with:

> “老师，‘有条不紊’的‘紊’是什么意思？”

### Tools

Repository | Language
---------- | --------
[vinta/paranoid-auto-spacing](https://github.com/vinta/paranoid-auto-spacing) | JavaScript
[serkodev/vue-pangu](https://github.com/serkodev/vue-pangu) | Vue.js (Web Converter)
[huei90/pangu.node](https://github.com/huei90/pangu.node) | Node.js
[huacnlee/auto-correct](https://github.com/huacnlee/auto-correct) | Ruby
[huacnlee/autocorrect](https://github.com/huacnlee/autocorrect) | Rust, WASM, CLI
[huacnlee/go-auto-correct](https://github.com/huacnlee/go-auto-correct) | Go
[sparanoid/space-lover](https://github.com/sparanoid/space-lover) | PHP (WordPress)
[nauxliu/auto-correct](https://github.com/NauxLiu/auto-correct) | PHP
[jxlwqq/chinese-typesetting](https://github.com/jxlwqq/chinese-typesetting) | PHP
[hotoo/pangu.vim](https://github.com/hotoo/pangu.vim) | Vim
[sparanoid/grunt-auto-spacing](https://github.com/sparanoid/grunt-auto-spacing) | Node.js (Grunt)
[hjiang/scripts/add-space-between-latin-and-cjk](https://github.com/hjiang/scripts/blob/master/add-space-between-latin-and-cjk) | Python
[hustcc/hint](https://github.com/hustcc/hint) | Python
[studygolang/autocorrect](https://github.com/studygolang/autocorrect) | Go
[n0vad3v/Tekorret](https://github.com/n0vad3v/Tekorrect) | Python
[VS Code - huacnlee.auto-correct](https://marketplace.visualstudio.com/items?itemName=huacnlee.auto-correct) | VS Code Extension

### References

- [Guidelines for Using Capital Letters - ThoughtCo.](https://www.thoughtco.com/guidelines-for-using-capital-letters-1691724)
- [Letter case - Wikipedia](https://en.wikipedia.org/wiki/Letter_case)
- [Punctuation - Oxford Dictionaries](https://en.oxforddictionaries.com/grammar/punctuation)
- [Punctuation - The Purdue OWL](https://owl.english.purdue.edu/owl/section/1/6/)
- [How to Use English Punctuation Correctly - wikiHow](https://www.wikihow.com/Use-English-Punctuation-Correctly)
- [格式 - openSUSE](https://zh.opensuse.org/index.php?title=Help:%E6%A0%BC%E5%BC%8F)
- [Halfwidth and fullwidth forms - Wikipedia](https://en.wikipedia.org/wiki/Halfwidth_and_fullwidth_forms)
- [引號 - 維基百科](https://zh.wikipedia.org/wiki/%E5%BC%95%E8%99%9F)
- [Interrobang - Wikipedia](https://en.wikipedia.org/wiki/Interrobang)
