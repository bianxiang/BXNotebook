Learning Responsive Web Design

[toc]

# 1 什么是响应式设计

# 2 响应式内容

With responsive websites, you need to think about content first, so youcan make sure your content will work well on small screens. If you’reusing existing content from a fixed-width website, you’re going to havea difficult time trying to shoehorn it into a layout for a smaller screen.If you’re starting from scratch with new content, you need to make sureit is optimized for any screen size, not just one screen size.

Content Strategy for the Web, Second Edition (http://contentstrategy.com/) by Kristina Halvorson and Melissa Rach (New Riders)

When redesigning an existing site, it’s a good first step to do a contentaudit, which is an inventory of all the content you currently have.
You can learn more about content audits by reading Donna Spencer’sUXmastery article, “How to Conduct a Content Audit (http://uxmastery.com/how-to-conduct-a-content-audit/).

# 3 响应式的HTML

# 4 响应式的CSSS

# （未）5 Media Queries

# 6 图片

{{粗读完，有空可以再细读。}}

基本上网站中所有的图片都会用到 `max-width`，因为直接全局设置：

```cssimg { max-width: 100%; }
```

Using max-width in IE 7 and earlier browsers creates some issues—the images don’t look good as they scale down. If you need to support those browsers, check out Ethan Marcotte’s blog post “fluid Images” (http://unstoppablerobotninja.com/entry/luid-images/) for a JavaScript solution.

Which responsive images solution should you use?” (http://css-tricks.com/which-responsive-images-solution-shouldyou-use/) on CSS-Tricks

Although you can’t actually use the `<picture>` element yet (because it hasn’t been implemented by the browsers), Scott Jehl has created a polyfill called Picturefill (https://github.com/scottjehl/picturefill) that essentially does the same thing using JavaScript.

HiSRC (https://github.com/teleject/hisrc) is a jQuery plug-in from Chris Schmitt that allows images to be replaced based on network speed and screen resolution.
The browser will first load a low-resolution “mobile first” version of each image. It will check the speed of the connection via JavaScript, and if the device has mobile bandwidth such as 3G, it will keep the lowres version of the image in place. If there’s more bandwidth available, it will download a higher-resolution version and replace the original low-res image with the new image file. If it also detects that the screen is high density, it will download and replace the image with an appropriate version.

You will also need to make and upload the three versions of each image.

You don’t have to use hiSrC for all the images on your site. If you don’t want to use it for a particular image, just use the `<img>` element without the extra `<div>`. another JavaScript plug-in that provides responsive images is foresight.js (https://github.com/adamdbradley/foresight.js).

Several companies offer responsive image services that will automaticallyresize your images to fit the screen width. As an example, we’ll look at Sencha.io SRC (http://www.sencha.com/learn/how-to-use-srcsencha-io/). This third-party service is free for small websites.

Other third-party services that provide responsive images include reSrC (http://www.resrc.it), Thumbr.io (http://www.thumbr.io), and responsive.io(https://responsive.io). Pricing is generally based on either the number ofimages or total GB per month.

For more details about how to choose image breakpoints, read JasonGrigsby’s “Sensible jumps in responsive image file sizes” (http://blog.cloudfour.com/sensible-jumps-in-responsive-image-file-sizes/) on theCloud Four Blog.

# 7 响应式工作流

{{粗读完，非常有空可以细读。}}

If you want to delve deeper into how to adjust your workflow to produceresponsive websites, check out Stephen Hay’s book Responsive DesignWorkflow (http://responsivedesignworkflow.com).

样式指引。A good place to start is the Style Guide Boilerplate (http://brettjankord.com/projects/style-guide-boilerplate/) from Brett Jankord.

Here are some examples of good web style guides of various types:- Starbucks.com (http://www.starbucks.com/static/reference/styleguide/) (style guide)
- South Tees Hospitals NHS Foundation Trust (http://www.southtees.nhs.uk/style-guide/) (style guide)
- Drupal.org (http://drupal.org/coding-standards) (coding standards)
- BBC’s digital services (http://www.bbc.co.uk/gel) (global experience language)
- Paul Robert Lloyd (http://www.paulrobertlloyd.com/about/styleguide/) (markup style guide)

Check out Dan rose’s “repurposing Photoshop for The Web” (http://www.smashingmagazine.com/2013/04/22/repurposing-photoshop/) on Smashing Magazine.

# （及以下未）8 Mobile and Beyond




