## MD预览
`marked`: 解析md文档的字符串为html字符串
`sanitize-html`: 过滤html字符串，以免引入危险代码

```tsx
import { marked } from 'marked';
import sanitizeHtml from 'sanitize-html';

export default function NotePreview({ children }: {
  children: string
}) {
  return(
    <div 
      className="text-with-markdown" 
      dangerouslySetInnerHTML={{
        __html: sanitizeHtml(marked(children) || '', 
        {
          allowedTags, 
          allowedAttributes
        })
      }}
    ></div>
  )
};
```

## 表单数据校验
`zod`