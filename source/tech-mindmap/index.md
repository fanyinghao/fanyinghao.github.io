---
title: tech-mindmap
date: 2018-07-12 22:22:08
---

<script type="text/javascript" src="https://cdn.rawgit.com/fex-team/kity/dev/dist/kity.min.js"></script>
<link href="https://cdn.rawgit.com/fex-team/kityminder-core/dev/dist/kityminder.core.css">
<script type="text/javascript" src="https://cdn.rawgit.com/fex-team/kityminder-core/dev/dist/kityminder.core.min.js"></script>

{% raw %}
<style type="text/css">
    .mindmap {
        width: 800px;
        height: 800px;
    }
</style>
{% endraw %}

{% pullquote mindmap %}
#主题
##一级分支
###二级分支
##一级分支
##一级分支
###二级分支
####三级分支
##一级分支
###二级分支
##一级分支
##一级分支
###二级分支
####三级分支 [hh](http://baidu.com)
##一级分支
###二级分支
##一级分支
##一级分支
###二级分支
####三级分支
##一级分支
###二级分支
##一级分支
##一级分支
###二级分支
####三级分支
##一级分支
###二级分支
##一级分支
##一级分支
###二级分支
####三级分支
##一级分支
###二级分支
##一级分支
##一级分支
###二级分支
####三级分支
##一级分支
###二级分支
##一级分支
##一级分支
###二级分支
####三级分支
{% endpullquote %}

{% raw %}
<script type="text/javascript">
setTimeout(function() {
        var minder = new kityminder.Minder({
            renderTo: '.mindmap'
        });
        var markdownText = $('.mindmap').text().trim();
        $('.mindmap p').each(function(index, element) {
            element.style.display = 'none';
            minder.execCommand('HyperLink', element.url, element.text);
        });
        minder.importData('markdown', markdownText);
        minder.disable();
        minder.execCommand('hand');
    },
    3000
)
</script>
{% endraw %}
