---
title: LinkedIn's Tips for Highly Effective Code Review
date: 2018-12-02 20:21:43
tags: CodeReview
categories: 翻译
toc: true
thumbnail: /gallery/startup-photos.jpg
---
[LinkedIn's Tips for Highly Effective Code 原文链接](https://thenewstack.io/linkedin-code-review/)

---
 LinkedIn recently passed the milestone of having conducted one million code reviews. The head of the social networking service’s tooling shared a few learned lessons along the way.

LinkedIn最近达成了一个新的里程碑，百万次的CodeReview。该社交网络服务工具的负责人分享了在此过程中的一些经验教训。

Reading and reviewing code is something every engineer does on a daily basis. A formal code review process, however, is a bit different — it requires every code change to be officially reviewed by another team member before the code goes to production. At LinkedIn, code reviews have been a mandatory part of our development process since 2011. Our goal in requiring code reviews was to scale our rapidly-growing engineering team as smoothly as possible. Good code reviews with meaningful, useful comments can really help with leveling up an entire engineering organization. At LinkedIn, these reviews have become an essential part of quality assurance and knowledge sharing. Embracing code reviews has changed our whole engineering culture for the better in several key ways.

阅读和审核代码是每个工程师日常工作的一部分。但是一个正统的CR过程还是稍有点不同的，它要求每次代码修改在进入正式产品前都要经过另外一个小组成员的正式审核。在LinkedIn，从2011年开始，CR就已经成为开发流程中必不可少的一步。我们引入CR的目的是尽可能平稳的扩张我们快速增长的工程师团队。好的CR包含有意义、有用的备注，它能切实的提升一个工程师组织的整体水平。在LinkedIn，CR也成为了QA和知识分享中必不可少的一部分。拥抱CR已经从多个关键方面改善了我们整个工程师文化。
<!--more-->
One of the biggest benefits of implementing company-wide code reviews has been increased standardization in our development workflow. Every team at LinkedIn uses the same tools and process for doing code reviews, which means that anyone can help with reviewing or contributing code for another team’s project. This eliminates problems like “I could fix the bug in their code, but how would I build that code and submit the fix?” This, in turn, helps increase collaboration across different teams in the engineering organization.

实现全公司范围的CR的一个最大的好处就是提升了我们开发流程中的标准化。在LinkedIn的每个小组都是用同样的工具和流程来进行CR，这意味着任何人都可以帮着审核或者贡献另一个项目的代码。这避免了这样的问题：“我知道怎么修复他们代码中的bug，但是我不知道怎么构建项目和如何提交修改”。换句话说，这也有助于一个工程师组织中跨小组的交流。

By making code reviews a mandatory process, we’ve also helped foster a healthy feedback culture at the company: engineers are open to giving and receiving feedback in all areas of work, not just in coding because it’s become a routine component of the job. Rather than viewing code reviews as critical or negative, our engineers use both giving and receiving code reviews as opportunities to grow professionally. In fact, high-quality code reviews are an important part of LinkedIn’s promotion process because they provide objective evidence of engineering skill.

通过将CR成为一个必要过程，我们也在公司培养了一种健康的反馈文化：正因为这成为了日常工作的一部分，所以工程师们更加开放的给出和接收工作上各方面的反馈，而不是仅仅局限于代码本身。我们的工程师CR当成是一次自我专业成长的机会，而不是觉得给别人做CR就是挑刺或者被别人做CR就是不开心的事。实际上，高质量的CR是LinkedIn晋升体系的一个重要部分，因为这本身就对工程师的技能提供了客观的证明。

Over the years, we’ve honed several best practices and tips for how to give truly great reviews. Below are some guidelines, in the form of questions, that we suggest asking to help make sure the reviewer and reviewee are getting the most value possible out of a code review.

经过这些年，关于怎么真正做好CR，我们总结出了一些最佳实践。下面是一些指导规范，通过一问一答的形式，我们希望确保审核者和被审核者都能达到一次CR的最佳效果。

### Do I Understand the “Why”?（我需要知道“为什么”吗？）

To facilitate the best review possible and help your team scale, every code change submission should include a design overview that briefly explains the motivation behind the change. It is really hard to offer a high-quality code review when the rationale needs to be inferred from the code change itself. It is fair to ask and expect the submitter to explain their motivation before attempting the code review. This also encourages the submitter to have an explanation in their commit message, increasing the quality of code documentation.

为了达到最佳的审核效果和帮助团队成长，每一次代码变更的提交都应该包含一份设计概要，这份设计概要应该解释清楚本次修改的动机。如果还要从代码本身去推测前因后果，这就很难去进行高质量的CR。在代码提交者发起CR前，要求他们解释代码修改的动机是很合理的。这也鼓励代码提交者在他们的提交备注里面加上具体的说明，以此提高代码文档的质量。

### Am I Giving Positive Feedback?（我应该给出正面的反馈吗？）

In an organization full of smart people, clean code and neat test coverage can be taken for granted. As a result, code review feedback tends to focus only on problems and issues found in the code. This is very unfortunate because most people need positive feedback to feel engaged and motivated — and engineers are no exception. When a reviewer sees good stuff in the code, they should call it out and give positive feedback. This helps improve team dynamics, and often such positive feedback is contagious. As with all code review comments (more on this below), any positive feedback should be specific, explaining why that particular code is well-written.

如果一个组织内都是睿智的工程师，那么干净整洁的代码和测试覆盖都是理所当然的。因此CR反馈倾向于只关注代码中的问题和缺陷。但是这又有点不公平，因为大多数人还是需要正面反馈，这样他们才能感受到鼓舞和激励，工程师也不例外。当一个审核者在代码中发现了一个好的范例时，他们应该指出来并且给予正面反馈。这有助于潜移默化的提升整个团队，通常来说这种正面反馈会慢慢传播开来。和所有的审核备注一样，任何正面反馈都应该明确指出为什么这部分特定代码写的棒。

### Is My Code Review Comment Explained Well?（我的CR备注解释清楚了吗？）

Whether the feedback is positive or negative, any code review comment should be self-explanatory. What might seem obvious to the reviewer can be unclear to an engineer who receives poorly-explained code review comments. When in doubt, it is better to over-explain than to provide terse feedback that yields more questions and the need for more back-and-forth communication. Explanations can be as simple as “reduces duplication,” “improves coverage,” or “makes code easier to test.” In addition to making reviewers’ comments clearer, these types of explanations also help reinforce the design principles that the team aspires to meet.

不管CR反馈是正面的还是负面的，任何CR备注都应该是自我解释的。如果被审核者只收到糟糕的不能自我解释的CR备注，那么他们可能就不清楚那些对审核者可能显而易见的东西。当遇到这种困惑时，宁可多写一些备注甚至过度解释，也比过于扼要的备注引入更多问题，然后需要更多来回交流要好。描述性的备注可以是简单的“减低重复”，“提高覆盖率”，或者“让代码更加容易测试”。除了让CR备注更清晰外，描述性的备注有助于再次加强团队追求的设计准则。

### Do I Appreciate the Submitter’s Effort?（我要欣赏提交者的努力吗？）

Hard work always needs to be appreciated, regardless of the outcome — this fosters strong, highly motivated teams. Some code changes are not of the highest quality and will need to be reworked. In those situations, it’s important to still acknowledge the effort that the author put into the changes, even if their code needs reworking. The best way to show appreciation is to put effort into your code review by giving high-quality feedback with decent explanations, acknowledging the good ideas (there are always good things in every code submission!), and using “thank you.”

不管结果如何，辛勤工作总是应该被欣赏的。这有助于培育一个强大的高效积极的团队。有些代码变更质量不高，并且需要返工。这种情况下，认可代码作者提交代码的努力依然很重要。表达欣赏的最佳方式是认真努力的做CR，给出高质量的反馈意见，包括详细的解释，指出好的建议（每次代码提交总归会有一些好东东），最后附上“谢谢”。

### Would This Review Comment Be Useful to Me?（这个CR备注对我有用吗？）

Asking this question is an easy and powerful way to validate if the code review comment is necessary. At the end of the day, engineers should view code reviews as helpful development tools, and not sources of unimportant busywork. If you don’t think a particular review comment would be useful to you, then remove it. A classic example of unhelpful code review comments are ones related to code formatting. Code style and formatting should be validated by automated tools, not engineers.

问这个问题是一个简单有效的用来鉴别CR备注是否必要的途径。最后，工程师们应该把CR当成是有用的开发工具而不是不重要的工作来源。如果你认为某个特定CR备注对你来说没有用，那么就删了它。一个经典的无用CR备注的例子就是那些关于代码格式的。代码风格和格式应该通过自动化工具而不是工程师来验证。

### Is the “Testing Done” Section Thorough Enough?（“通过测试”章节足够吗？）

At LinkedIn, every code change submission has a mandatory “testing done” section that needs to be filled out. In the open source world, using GitHub as an example, engineers can submit “testing done” information in the pull request description. What should be in “testing done” depends on the gravity of the changes and the current level of test coverage. If the change contains new or altered conditional complexity, it is fair to expect it to be covered in unit tests. Some changes may require running manual testing if the integration test coverage is inadequate. In those cases, “testing done” should include information about the test scenarios and the outputs. When changes alter the output of the program, it is very useful to include the new output in the “testing done” section.

在LinkedIn，每次代码修改提交都有一个必填的“通过测试”章节需要填写。在开源世界里，拿GitHub来举例，工程师可以在Pull Request说明里面提交“通过测试”信息。至于哪些信息必须包含在“通过测试”章节里面，取决于这次修改的重量级和当前测试覆盖率水平。如果提交包含新的条件或者改变了原有的条件复杂度，那么很合理的希望在Unit test中能覆盖这部分修改。如果集成测试覆盖率不足的时候，有些修改可能需要进行人工测试。在这种情况下，“通过测试”章节应该包括测试用例和输出等信息。当代码修改改变了程序的输出结果，那么在“通过测试”章节里面加上新引入的程序输出也是很有帮助的。

### Am I Too Pedantic in My Review?（我会不会在CR中太学究了？）

Some code reviews have so many comments that important issues — ones that really need to be fixed — are lost among less important suggestions. Reviews that are too heavy on details for a given team can slow down the review cycle and cause friction for both the reviewers and reviewee. Having clear review expectations, example reviews, and a positive, inviting review culture are great ways to avoid lengthy, exhausting review cycles.

有些CR中包含太多的CR备注了以至于重要的问题（那些真正需要修复的）被淹没在那些不那么重要的意见中。一个团队中，在细节上过于笨重的CR会减速整个Review闭环，也会潜在的引起审核者和被审核者双方的摩擦。保持清晰的Review说明，Review示范以及正面，吸引人的Review文化是避免冗长的让人精疲力尽的Review闭环的良好途径。

In summary, having a formal code review process helps improve code quality, team learning, and knowledge sharing. The quality of work increases when every engineer on the team realizes two important things: someone else will read my code, so it better be good, and I’ll have to address any review comments I receive, so I should try to make my code good the first time around to save myself effort later on. When code reviews become an everyday habit, the team practices giving and receiving feedback on a daily basis. This is key for growth and improvement.

总之，通过正统的CR流程可以帮助提高代码质量，团队学习以及知识分享。工作质量的提升是顺理成章的事，只要团队中的每个工程师都意识到这两点：有人会看我的代码，所以最好写好一点。并且我不得不回复任何我收到的CR备注，所以最好就是第一次就把代码写好点，这样后面我还能省点力气。当CR成为每天的习惯时，团队也就在每天的日常中给出和得到反馈。这是团队成长和提升的关键所在。

At LinkedIn, we’ve learned a lot from the past one million code reviews, and we aspire to learn even more from the next million. The more effort that’s put into code reviews, the better a team gets at giving great code reviews, the higher the quality of code that is checked in, and the higher the quality of the product that’s built. High-quality code reviews are contagious!

在LinkedIn，我们从过去百万的CR中收获良多，也希望能从下一个百万CR中收获更多。你在CR上投入的努力越多，你的团队就越能做好一个优秀的CR。你提交的代码质量越高，构建的产品的质量就越高。高质量CR是有生命力和感染力的。

_The author thanks James Miller, Oscar Bonilla, Joshua Olson, Andrew Macleod, Scott Meyer, and Deep Majumder for insightful feedback on this article._

_LinkedIn’s parent company, Microsoft, is a sponsor of The New Stack._

_Feature image by JJ Ying on Unsplash._