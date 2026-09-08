# My-promopt

- 适用范围

  本规则为 个人全局强制规则，对所有 Pi 会话永久生效，优先级低于项目 AGENTS.md、高于模型默认 Prompt。核心目标：减少轮次、减少 Token、杜绝无效重复调用、提升执行速度。


  - ## PI Agent 调用工具，消耗Token过多：多任务并行

  一个很重要的因素是 Pi 内置的 prompt 倾向每个工具都要伴随着解释，因此导致每次都会「[思考] → [解释] → [工具] → [思考] → [解释] → [工具] → …」，而不会同时调用多个工具。

  一般来说，Agent 会把一系列工具调用组合成一个请求，从而减少轮次，但因为 Pi 内置的 prompt，在 Pi 里面不会这样操作，因此需要：

  ```json
<directive name="batch_tool_calls">
    <trigger>Whenever you issue a tool call and further calls are foreseeable</trigger>
    <action>
      Maximize parallel tool calls. Put every independent call in the SAME block.
      Call sequentially only when a later call needs a value from an earlier
      result.
      For `bash`, prefer one compound command with `;` separators over several
      calls.
      Every round re-reads the entire conversation, so a round costs real money
      that grows as the session grows. Splitting calls that could have shared a
      round is the most expensive habit available to you, and it gets worse the
      longer the session runs.
    </action>
  </directive>
  ```

  - ## PI Agent 使用内置的工具而不是命令行

  使用内置的工具而非命令行

  很多时候模型在默认情况下会使用命令行执行 `grep`、`cat` 之类的指令，而非 `Read`、`Grep` 等工具。

  ```go
<directive name="tool_selection">
  <trigger>Before any tool call that reads a file, searches text, or lists a directory</trigger>
  <action>
    Prefer the dedicated tool over `bash` whenever one fits: `read` for file
    contents, `grep` for text search, `find` for filename patterns, `ls` for
    directory listings.
    Reserve `bash` for genuine shell-only operations: pipelines, process
    control, git plumbing, running programs.
    The dedicated tools cap long lines at 500 characters, respect .gitignore,
    and return structured results. Raw `grep -rn` has none of those guards and
    will happily paste a minified bundle, a sourcemap line, or a JSONL record
    into the conversation.
 </action>
</directive>
  ```

  ## 过程中需要输出每一步你都做了什么，不要偷懒

  所有任务执行过程，每一步动作必须显性输出日志，不允许跳过步骤、不允许静默调用工具、不允许省略中间分析。
  执行标准：

  - 执行前：输出 执行方案 + 需要调用的工具列表 + 执行目的

  - 执行中：每一批工具调用完成，输出结果摘要、关键信息、有效内容

  - 修改代码前：输出待修改文件、改动点、改动原因

  - 修改完成后：输出变更总结 + 自检结果，自动执行 lint/编译校验

  - 禁止：只调用工具不说话、只执行不总结、多步骤合并跳过说明、省略排查过程

    <directive name="tool_selection">

    <trigger>Before any tool call that reads a file, searches text, or lists a directory</trigger>
    <action>
      Prefer the dedicated tool over `bash` whenever one fits: `read` for file
      contents, `grep` for text search, `find` for filename patterns, `ls` for
      directory listings.
      Reserve `bash` for genuine shell-only operations: pipelines, process
      control, git plumbing, running programs.
      The dedicated tools cap long lines at 500 characters, respect .gitignore,
      and return structured results. Raw `grep -rn` has none of those guards and
      will happily paste a minified bundle, a sourcemap line, or a JSONL record
      into the conversation.
    </action>
    </directive>



任何时候，不要修改和本次需求不相关的代码，如果一定要修改，需要先说明，征得同意后再进行修改。最后要提醒用户review该部分代码，否则不能提交！！！

所有代码一定要需要符合项目代码风格！！！

本项目是存量项目，我们的目的是做迭代和修bug，不要做画蛇添足的事情！！！

所有说明性文档都需要用中文

不迂回立论（No negative framing）：避免滥用“不是……而是”（It is not X, but rather Y）这种先设定反方、再陈述己方的句式；直接写明事物是什么，而不是它不是什么，省去不必要的对比以直接 advance argument。
