---
layout: post
title: Running chapter headers in Microsoft Word
date: '2012-03-18 15:31:42'
tags:
- ms-word
---

I’m in the process of writing up my doctoral thesis, and have had to wrestle with a lot of Word formatting. The biggest difficulty I had was separating the whole document into chapters. I wanted each chapter to start with a big heading like “Chapter 1 Introduction”, and to have that title repeated in the header of each subsequent page; i.e. running chapter headers throughout the document. It’s quite fiddly to do, but very useful once you’ve got it setup.

At the time of writing, I’m still working in the ancient Word 2003. The same principles seem to work in later versions as well. The only differences are usually in the menu structure.

## Section breaks

This is the most important part of the process. You need to insert a section break between each chapter of your document so that Word knows how your layout works. You can optionally make it a “continuous” break which means the chapters run directly together, or a “next page” break which means the new chapter starts on a new page.

- **Word 2003:** Click the “Insert” menu then “Break…”
- **Word 2007/2010:** Click the “Page Layout” ribbon tab then “Breaks”

## Use a heading style

I recommend using the built-in “Heading 1” style for your chapter headings. You can modify it or use a different named style if you want. Either way, make sure you are consistent and that you don’t use the same style for anything else.

If you’re not familiar with styles in Word then I really recommend learning about them. They can make formatting much easier. For some information, see: [Customize or create new styles](https://support.office.com/en-us/article/customize-or-create-new-styles-d38d6e47-f6fc-48eb-a607-1eb120dec563)

## Different first-page headers

You usually don’t want the running header to appear on the first page of a chapter as you already have the heading there anyway. There are a couple of ways to do this, but my preferred approach is to tell Word that the first page of each section will have a different header.

Click on the first page of your first chapter, and then open the “Page Setup” dialog:

- **Word 2003:** Click the “File” menu then “Page Setup”.
- **Word 2007/2010:** Click the “Page Layout” ribbon tab, then click the little arrow at the bottom-right of the “Page Setup” group.

When the “Page Setup” dialog is open, click on the “Layout” tab. Under the “Headers and footers” section, check the box labelled “Different first page”, then click OK.

You will need to repeat this process for each chapter.

## Add the running header

Finally, we can add our running header. Open the header on the 2nd page or later of any chapter. Now you need to insert a field which will automatically display your chapter heading. To do this, open the “Field” dialog:

- **Word 2003:** Click the “Insert” menu and then “Field…”
- **Word 2007/2010:** Click on the “Insert” ribbon tab, then “Quick Parts” (in the Text group), then “Fields”

The Field dialog should have a “Categories” drop-down box at the top-left. Select the “(All)” category. In the “Field names” box underneath it, scroll down and click on “StyleRef”. Over on the right, a box labelled “Style names” should appear. Click on the name of the style you used for chapter headings, such as “Heading 1”.

Click on OK, and your running chapter heading should now appear throughout your document.

## Tips and problems

### How do I include heading numbers?

If you are using automated heading numbering then the default StyleRef field probably won’t show it. Follow the same process to insert a second StyleRef field into your header. However, don’t click OK on the Field dialog just yet. First, check the box labelled “Insert paragraph number”, and then click OK.

That should insert the number as a separate field. You can move or copy-paste it to wherever you want in the header.

### The running title is missing on some pages

If you find that the running header stops at some point in your document then you may have broken the link between headers. Open up the first header which is causing a problem and try enabling “Link to previous”.

If that doesn’t work (or you deliberately disabled “Link to previous”) then you could try just copy-pasting a working header in there. The field code should automatically update to display the suitable running header wherever you paste it.

### I changed a chapter title, but the running header still shows the old title

Open the header, click on the title, and press F9. That should tell Word to update the field.

### The running header isn’t displaying a chapter title

Make sure you aren’t using the chapter title style for anything else. The StyleRef field looks for the latest instance of the specified style so using it for other purposes will cause problems.

<!--kg-card-end: markdown-->