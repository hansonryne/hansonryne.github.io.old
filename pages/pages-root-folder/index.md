---
#
# Use the widgets beneath and the content will be
# inserted automagically in the webpage. To make
# this work, you have to use › layout: frontpage
#
layout: frontpage
header:
  image_fullwidth: header_unsplash_12.jpg
widget1:
  title: "Blog & Portfolio"
  url: 'https://www.rynehanson.com/blog/'
  image: widget-1-302x182.jpg
  text: 'Browse the blog.  It is updated with content every so often.  :)'
widget2:
  title: "Check out my GitHub"
  url: 'https://github.com/hansonryne'
  text: 'A sample of some public projects I have started and mostly not finished.'
  image: https://github.githubassets.com/images/modules/logos_page/GitHub-Mark.png
widget3:
  title: "Follow me on Twitter"
  url: 'https://twitter.com/_hansonet_'
  image: https://www.sketchappsources.com/resources/source-image/twitterlogo_1x.png
  text: 'Twitter is my main social media presence.  Its where I keep up to date.'
#
# Use the call for action to show a button on the frontpage
#
# To make internal links, just use a permalink like this
# url: /getting-started/
#
# To style the button in different colors, use no value
# to use the main color or success, alert or secondary.
# To change colors see sass/_01_settings_colors.scss
#
#callforaction:
#  url: https://tinyletter.com/feeling-responsive
#  text: Inform me about new updates and features ›
#  style: alert
# permalink: /index.html
#
# This is a nasty hack to make the navigation highlight
# this page as active in the topbar navigation
#
homepage: true
---

<div id="videoModal" class="reveal-modal large" data-reveal="">
  <div class="flex-video widescreen vimeo" style="display: block;">
    <iframe width="1280" height="720" src="https://www.youtube.com/embed/3b5zCFSmVvU" frameborder="0" allowfullscreen></iframe>
  </div>
  <a class="close-reveal-modal">&#215;</a>
</div>
