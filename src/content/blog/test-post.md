---
title: "Test Post with All Features"
description: "A comprehensive test post demonstrating code blocks, quotations, bullet points, and images"
pubDate: 2025-01-15
tags: ["test", "features", "demo"]
---

# Test Post with All Features

This is a test post that demonstrates all the key features of the blog: code blocks, quotations, bullet points, and images.

## Code Blocks

Here's a Python example with a function:

```python
def calculate_fibonacci(n):
    """Calculate the nth Fibonacci number."""
    if n <= 1:
        return n
    a, b = 0, 1
    for _ in range(2, n + 1):
        a, b = b, a + b
    return b

# Example usage
result = calculate_fibonacci(10)
print(f"The 10th Fibonacci number is: {result}")
```

And here's some JavaScript/TypeScript:

```typescript
interface BlogPost {
  title: string;
  content: string;
  published: boolean;
}

const createPost = (title: string, content: string): BlogPost => {
  return {
    title,
    content,
    published: true,
  };
};

const myPost = createPost("Hello World", "This is my first post!");
console.log(myPost);
```

## Quotations

Here are some inspiring quotes:

> The best way to predict the future is to invent it.
> 
> — Alan Kay

> Code is like humor. When you have to explain it, it's bad.
> 
> — Cory House

> Simplicity is the ultimate sophistication.
> 
> — Leonardo da Vinci

## Bullet Points

### Features of this blog:

- **Cream background** for comfortable reading
- **Syntax highlighting** with Shiki
- **Markdown support** for easy writing
- **Responsive design** that works on all devices
- **Fast loading** with static site generation
- **Zero backend** complexity

### Technologies used:

- Astro for the framework
- Markdown for content
- Shiki for code highlighting
- Vanilla CSS for styling

## Numbered Lists

Here's how to write a blog post:

1. Create a new `.md` file in `src/content/blog/`
2. Add frontmatter with title, description, and date
3. Write your content in Markdown
4. Save the file
5. The post will automatically appear on your blog!

## Mixed Content

You can combine different elements:

- Use **bold text** for emphasis
- Add `inline code` when needed
- Mix *italics* with other formatting
- Include [links](https://astro.build) naturally

### Code in Lists

Here are some useful commands:

- `npm run dev` - Start development server
- `npm run build` - Build for production
- `npm run preview` - Preview production build

## Images

![Astro Logo](https://astro.build/assets/press/astro-logo-dark.svg)

*Note: In production, you'd typically place images in the `public/` folder and reference them like `/image-name.png`*

You can also use local images by placing them in the `public/` directory:

```markdown
![Description](/path/to/image.png)
```

## Conclusion

This test post demonstrates:

- ✅ Code blocks with syntax highlighting
- ✅ Blockquotes with proper styling
- ✅ Bullet points and numbered lists
- ✅ Image support
- ✅ All Markdown features working together

Happy blogging!

