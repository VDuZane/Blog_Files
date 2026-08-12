# VDu Blog_Files 目录模板

把本目录内容拷到私有仓 [VDuZane/Blog_Files](https://github.com/VDuZane/Blog_Files) 作为初始结构。

```text
posts/           # 文章 Markdown（GithubPush / 手工写作）
  tech/
  novel/
media/           # 图片/音视频（按文章 slug 分子目录）
pages/           # 独立页面
_meta/           # 分类标签说明（可选）
```

## 文章示例 front matter

见 `posts/tech/hello-github-primary.md`。

## 双向流程（推荐）

1. **以 GitHub 为主**：在仓库或本地改 `posts/**/*.md`，再同步到 Typecho（GithubSync / Actions）。  
2. **网页补写/修订**：Typecho 保存后由 **GithubPush** 写回同路径文件。  
3. 打 `git tag` 做版本快照。
