حسناً، دعني أفهم المشكلة من الأساس بدلاً من التخمين.الآن وجدت المشكلة الحقيقية بالضبط. `threadedCommentJs` يستدعي `blogger.widgets.blog.initThreadedComments()` — هذا هو النظام الصحيح والحديث. لكن المشكلة أن:

**`threadedCommentForm`** لا يزال يحتوي على الـ iframe القديم `BLOG_CMT_createIframe` **ولم يُستبدل**.

المشكلة ليست في JS — المشكلة أن **نموذج التعليق** `threadedCommentForm` يُولِّد iframe فارغاً لأن `BLOG_CMT_createIframe` لا يعمل مع النظام الجديد. والنظام الجديد `initThreadedComments` يبني واجهة التعليق بنفسه داخل `#comment-holder` — **لكنه يحتاج أن يُشغَّل دائماً** وليس فقط عند `embedCommentForm`.

إليك **الكود النهائي المضمون** — فقط ثلاثة تغييرات محددة في الملف:

---

**التغيير 1:** في `threadedComments` — السطر 4743، احذف شرط `embedCommentForm`:

```xml
<!-- قبل -->
<b:if cond='data:post.embedCommentForm'>
  <b:include data='post' name='threadedCommentJs'/>
</b:if>

<!-- بعد -->
<b:include data='post' name='threadedCommentJs'/>
```

---

**التغيير 2:** استبدل `threadedCommentForm` بالكامل:

```xml
<!-- قبل (السطر 4703-4722) -->
<b:includable id='threadedCommentForm' var='post'>
  <div class='comment-form'>
    <a name='comment-form'/>
    <h4 id='comment-post-message'>
      <data:messages.postAComment/>
    </h4>
    <b:if cond='data:this.messages.blogComment != &quot;&quot;'>
      <p>
        <data:this.messages.blogComment/>
      </p>
    </b:if>
    <b:include data='post' name='commentFormIframeSrc'/>
    <iframe allowtransparency='allowtransparency' class='blogger-iframe-colorize blogger-comment-from-post' expr:height='data:cmtIframeInitialHeight ?: &quot;90px&quot;' frameborder='0' id='comment-editor' name='comment-editor' src='' width='100%'/>
    <data:post.cmtfpIframe/>
    <script type='text/javascript'>
BLOG_CMT_createIframe(&#39;<data:post.appRpcRelayPath/>
&#39;);
    </script>
  </div>
</b:includable>
```

```xml
<!-- بعد -->
<b:includable id='threadedCommentForm' var='post'>
  <div class='arwn-cmt-form'/>
</b:includable>
```

نعم — فارغ تماماً. لأن `initThreadedComments` يبني نموذج الإدخال بنفسه داخل `#comment-holder`. الـ iframe القديم هو ما كان يتداخل معه ويمنعه.

---

**التغيير 3:** أضف هذا CSS في قسم `<style>`:

```css
/* إصلاح قسم التعليقات */
p.comment-footer { display: block !important; }
#comments.embed[data-num-comments="0"] p.comment-footer { display: block !important; }
#comments { border-top: 1px dashed var(--border-color) !important; }
#comment-editor-src { display: none !important; }
```

---

هذا هو الحل الصحيح لأن `initThreadedComments` هو المسؤول عن بناء نموذج الإدخال — وكل ما نحتاجه هو تشغيله دائماً وإزالة الـ iframe القديم الذي كان يتعارض معه.