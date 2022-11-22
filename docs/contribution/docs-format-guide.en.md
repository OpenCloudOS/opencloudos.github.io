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
    The "Chinese Copywriting Guidelines" referenced below, and written by GitHub user sparanoid, is licensed under the MIT license. [GitHub repository](https://github.com/sparanoid/chinese-copywriting-guidelines)

### Spacing

> Research shows that, people adding no space between Chinese and English suffer from pathetic relationships. 70% of them are married by the age of 34, with someone they don't love; 30% of them left everything for their cats and died. Blank spaces are essential to both romance and writing.
>
> 與大家共勉之。」——[vinta/paranoid-auto-spacing](https://github.com/vinta/pangu.js)

#### Place one space before/after English words

Good:

> 在 LeanCloud 上，數據儲存是圍繞 `AVObject` 進行的。

Bad:

> 在 LeanCloud 上，數據儲存是圍繞`AVObject`進行的。
>
> 在 LeanCloud 上，數據儲存是圍繞`AVObject` 進行的。

An example of complete and correct usage:

> 在 LeanCloud 上，數據儲存是圍繞 `AVObject` 進行的。每個 `AVObject` 都包含了與 JSON 兼容的 key-value 對應的數據。數據是 schema-free 的，你不需要在每個 `AVObject` 上提前指定存在哪些键，只要直接設定對應的 key-value 即可。

Exceptions: For product and brand names, please refer to the writing format of the official definition. For example, use “豆瓣FM” instead of “豆瓣 FM”.

#### Place one space before/after numbers

Good:

> 今天出去買菜花了 5000 元。

Bad:

> 今天出去買菜花了 5000 元。
>
> 今天出去買菜花了 5000 元。

#### Place one space between numbers and units

Good:

> 我家的光纖入屋寬頻有 10 Gbps，SSD 一共有 20 TB。

Bad:

> 我家的光纖入屋寬頻有 10Gbps，SSD 一共有 20TB。

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

> 剛剛買了一部 iPhone，好開心！

Bad:

> 剛剛買了一部 iPhone，好開心！
>
> 剛剛買了一部 iPhone，好開心！

#### `text-spacing` to the rescue?

[`text-spacing`](https://www.w3.org/TR/css-text-4/#text-spacing-property) and [`-ms-text-autospace`](https://msdn.microsoft.com/library/ms531164(v=vs.85).aspx) provided by CSS Text Module Level and Microsoft can specify the autospacing and narrow space width adjustment of text. However it's not popular, and on other platforms such as OS X and iOS we can not use this feature. So it's better for you to keep up the habit.

### Punctuation

#### Avoid duplicate punctuation

Although the punctuation usage of China mainland admits to duplicate the punctuations, the sentence may become unpleasing to the eye by doing that.

Good:

> 德國隊竟然戰勝了巴西隊！
>
> 她竟然對你說「喵」？！

Bad:

> 德國隊竟然戰勝了巴西隊！！
>
> 德國隊竟然戰勝了巴西隊！！！！！！！！
>
> 她竟然對你說「喵」？？！！
>
> 她竟然對你說「喵」？！？！？？！！

### Fullwidth and halfwidth

If you're not familiar with fullwidth and halfwidth forms please refer to article [Halfwidth and fullwidth](https://en.wikipedia.org/wiki/Halfwidth_and_fullwidth_forms) on Wikipedia.

#### Use punctuation in fullwidth form

Good:

> 嗨！你知道嘛？今天前台的小妹跟我說「喵」了哎！
>
> 核磁共振成像（NMRI）是什麼原理都不知道？JFGI！

Bad:

> 嗨！你知道嘛？今天前台的小妹跟我說 "喵" 了哎！
>
> 嗨！你知道嘛？今天前台的小妹跟我說"喵"了哎！
>
> 核磁共振成像 (NMRI) 是什麼原理都不知道？JFGI!
>
> 核磁共振成像 (NMRI) 是什麼原理都不知道？JFGI!

#### Use numbers in halfwidth form

Good:

> 這件蛋糕只賣 1000 元。

Bad:

> 這件蛋糕只賣 1000 元。

Exceptions: fullwidth numbers are acceptable for better visual alignment in graphic design.

#### Use punctuation in halfwidth form for English sentences

Good:

> 賈伯斯那句話是怎麼說的？「Stay hungry, stay foolish.」
>
> 推薦你閱讀《Hackers & Painters: Big Ideas from the Computer Age》，非常的有趣。

Bad:

> 賈伯斯那句話是怎麼說的？「Stay hungry，stay foolish。」
>
> 推薦你閱讀《Hackers＆Painters：Big Ideas from the Computer Age》，非常的有趣。

### Nouns

#### Use correct case for proper nouns

The case usage of proper nouns is related to English writing, and is not the topic of this wiki. So only some common mistakes are listed here.

Good:

> 使用 GitHub 登錄
>
> 我們的客戶有 GitHub、Foursquare、Microsoft Corporation、Google、Facebook, Inc.。

Bad:

> 使用 github 登錄
>
> 使用 GITHUB 登錄
>
> 使用 Github 登錄
>
> 使用 gitHub 登錄
>
> 使用 g ｲんĤЦ8 登錄
>
> 我們的客戶有 github、foursquare、microsoft corporation、google、facebook, inc.。
>
> 我們的客戶有 GITHUB、FOURSQUARE、MICROSOFT CORPORATION、GOOGLE、FACEBOOK, INC.。
>
> 我們的客戶有 Github、FourSquare、MicroSoft Corporation、Google、FaceBook, Inc.。
>
> 我們的客戶有 gitHub、fourSquare、microSoft Corporation、google、faceBook, Inc.。
>
> 我們的客戶有 g ｲんĤЦ8、ｷ ouЯƧqu ﾑгє、๓เςг๏ร๏Ŧt ς๏гק๏гคtเ๏ภn、900913、ƒ4 ᄃëв๏๏к, IПᄃ.。

Please note that when the text needs to be displayed in all uppercase or all lowercase for visual consistency, please use the standard case in HTML and use `text-transform: uppercase;`/`text-transform: lowercase;` to define the presentation.

#### Avoid jargons

Good:

> 我們需要一位熟悉 TypeScript、HTML5，至少理解一種框架（如 React、Next.js）的前端開發者。

Bad:

> 我們需要一位熟悉 Ts、h5，至少理解一種框架（如 RJS、nextjs）的 FED。

### Dispute

The following usages comprise of personal characteristics. As such, from the perspective of copywriting guidelines, they are **still correct** regardless of whether they comply with the following rules.

#### Add extra spaces before/after links

!!! note
    In this case, we recommend you to treat the link content as normal content, that is, if the content before and after the link is in Chinese and the link name is in Chinese, then you do not need to add spaces before and after the link. Otherwise, add spaces between English/digits and Chinese.

Usage:

> 請 [提交一个 issue](#) 並分配给相關同事。
>
> 訪問我們網站的最新動態，請 [點擊這裡](#) 進行訂閱！

compared with:

> 請[提交一个 issue](#) 並分配给相關同事。
>
> 訪問我們網站的最新動態，請[點擊這裡](#)進行訂閱！

#### Use corner brackets for Chinese Simplified

!!! note
    We recommend the use of the corner brackets.

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
