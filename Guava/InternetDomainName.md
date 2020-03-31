## 介绍

`InternetDomainName`是一个用于解析和操作域名的工具。它可以用作验证器、组件提取器和值类型，以便以类型安全的方式传递域名。

然而，`InternetDomainName`行为在某些方面可能会令人吃惊，并可能导致调用代码的错误。本文档解决了这些问题。

## 详情

### 公共后缀和私有域

根据相关的RFC规范，可以保证`InternetDomainName`对象在语法上是有效的，但不能保证它与`Internet`上的实际可寻址域相对应。如果不进行域的网络查找并尝试与它联系，就不可能做到这一点，对于大多数情况，这是无法接受的开销。

不过，确定给定的域名是否代表`Internet`上的实际域名通常非常有用。为此，我们使用来自公共后缀列表[Public Suffix List (PSL)](http://publicsuffix.org/)的数据，该列表由`Mozilla`基金会维护。在`InternetDomainName`上有一些方法来确定给定域与`PSL`之间的关系。用最基本的术语来说，如果`domain.haspublicsuffix()`返回`true`，则该域可能对应一个实际的`Internet`地址;否则，它几乎肯定不会。

此时，我们需要备份并定义一些术语。有四个兴趣条款:

* [Top-Level Domain (TLD)](http://en.wikipedia.org/wiki/Top_level_domain): 没有子域名的单标签域名，如com或au。
* Registry suffix:由域名注册中心(如Verisign for com)控制的域名(如com或co.uk)，在该域名注册中心下，人们可以通过域名注册中心(如Namecheap)注册子域名。此类域名注册受到`ICANN`等互联网管理机构的法律保护。
* [Public suffix](http://en.wikipedia.org/wiki/Public_Suffix_List):这个类别是注册表后缀的超集，此外还包括`blogspot.com`等后缀，这些后缀不受注册控制，但允许公众注册子域名。在一些常见的情况下，使用公共后缀而不是注册表后缀对域进行分类更合适。例如，永远不要将`cookie`设置为公共后缀。
* Effective Top-Level Domain:不赞成的同义词公共后缀。

在继续之前，有必要仔细阅读相关文章。

造成混淆的一个主要原因是，当人们说“TLD”时，他们的意思是注册表后缀或公共后缀。这三个都是独立的概念。举个例子,

* com是全部三个:TLD、注册表后缀和公共后缀
* au是TLD，但不是注册表后缀和公共后缀
* com.au是注册表后缀和公共后缀，但不是TLD
* blogspot.com是一个公共后缀，但既不是注册表后缀，也不是TLD
* squerf不是这三个中的任何一个

这种混淆尤其危险，因为TLD和注册表后缀有清晰、正式的定义，而公共后缀没有。最后，一个可信的来源要求PSL维护人员将一个公共后缀加到列表中。可信的消息来源包括ICANN和国家域名管理器，但也包括提供服务的私营公司，这些服务具有(模糊地)定义公共后缀的特征——独立子域名和超级cookie抑制。因此，许多google拥有的域(例如`blogspot.com`)都包含在PSL中。

回到`InternetDomainName`，只要我们限制自己使用`hasPublicSuffix()`来验证该域是一个合理的`Internet`域，一切都很好。危险来自于识别或提取“顶级私有域”的方法。从技术角度看，顶部的私有域只是公共后缀之前最右边的超域。例如，`www.foo.co.uk`有一个`co.uk`的公共后缀和一个`foo.co.uk`的顶级私有域。

正如`isUnderPublicSuffix()`、`isTopPrivateDomain()`和`topPrivateDomain()`的文档中所指出的，这些方法(大多数情况下)惟一可靠的功能是确定可以在何处设置`cookie`。然而，许多人实际上试图做的是从子域中找到“real”域或“owner”域。例如，在`mail.google.com`中，他们希望将`google.com`标识为所有者域。所以他们写:

```java
InternetDomainName owner =
    InternetDomainName.from("mail.google.com").topPrivateDomain();
```

…果不其然，域名的所有者最终是`google.com`。事实上，这个习语(以及类似的习语)在很多时候都很管用。“the domain under the public suffix”在语义上应该等同于“the owner domain”，这似乎很直观。

但事实并非如此，这就是问题所在。考虑`blogspot.com`，它出现在PSL中。虽然它具有公共后缀的特点——人们可以注册域名下(博客),和cookies不应该允许设置(防止cross-blog cookie恶作剧),它本身就是一个可寻址域在互联网上(这发生在重定向到`blogger.com`,但这很容易改变)。

因此，如果在`foo.blogspot.com`上使用上面的习惯用法，所有者将是相同的域`foo.blogspot.com`。对于许多常见的应用程序来说，这是设置cookie的正确答案，但这显然会让许多人感到惊讶。

对于那些目标实际上是确定从注册中心购买的域名的罕见应用程序，正确的抽象不是公共后缀，而是注册表后缀。如果我们改变上面的代码，使用注册表后缀的并行方法:

```java
InternetDomainName owner =
    InternetDomainName.from("foo.blogspot.com").topDomainUnderRegistrySuffix();
```

…最后是`owner`设置为`blogspot.com`。

这个方便的表格总结了不同类型后缀的区别:

| InternetDomainName | publicSuffix() | topPrivateDomain() | registrySuffix() | topDomainUnderRegistrySuffix() |
| - | - | - | - | - |
| google.com | com | google.com | com | google.com |
| co.uk | co.uk | N/A | co.uk | N/A |
| www.google.co.uk | co.uk | google.co.uk | co.uk | google.co.uk |
| foo.blogspot.com | blogspot.com | foo.blogspot.com | com | blogspot.com |
| google.invalid | N/A | N/A | N/A | N/A |

这里最大的教训是:

* TLDs, 注册表后缀, 公共后缀不是一回事。
* 公共后缀由人类定义，用于严格限制的目的(主要是域验证和超级cookie预防)，并且不可预测地更改。
* 给定域与公共后缀的关系、该域响应web请求的能力以及该域的“所有权”之间没有定义映射。
* 注册表后缀可以帮助您确定域的“所有权”，即icann样式的域注册;但是，对于大多数应用程序，此信息不如公共后缀合适。
* 您可以使用`InternetDomainName`来确定给定的字符串是否表示`Internet`上的可寻址域，以及确定域的哪些部分可能允许设置cookie。
* 您不能使用`InternetDomainName`来确定一个域是否作为可寻址主机存在于`Internet`上。

请记住，如果您不注意这个建议，您的代码将在各种各样的输入上运行……但是，失败案例都是等待发生的bug，随着PSL更新被合并到底层`InternetDomainName`代码中，失败案例集将发生更改。

