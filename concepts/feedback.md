  @ACCESSIBILITY : How would you go about making this application more accessible those with physical disabilities or who are visually impaired? What changes would need to be made and what are the best practices in Rails and Jquery for using Aria attributes?
    To put it briefly, I would convert my app into something more dynamic, so that I could take advantage of the `role="alert"` attribute for errors and as feedback while the user is typing if the user exceeds 500 characters. Additionally, I would set tabindexes on several of the main elements (nav, headers, textarea) so that these main elements serve as landmarks and the page is easier to navigate for the user. Finally, I would add an `aria-label` for the textarea, briefly describing what it is and the character limit. The default semantics for most of these elements takes care of the rest. 

      I) Aria attributes -- These to make content more navigable and understandable. I add Aria attributes to HTML elements so that screen readers can pick them up and keyboard users can navigate to these elements. More specifically: "WAI-ARIA, the Accessible Rich Internet Applications Suite, defines a way to make Web content and Web applications more accessible to people with disabilities. It especially helps with dynamic content and advanced user interface controls developed with Ajax, HTML, JavaScript, and related technologies." (https://www.w3.org/WAI/intro/aria)

      II) jQuery --
        - Errors & Submission: I would use jQuery in combination with Aria attributes to help accessibility. I would convert my app from a static app to a dynamic one, using jQuery and AJAX to submit the form without refreshing the page. The benefit of doing so would be for screen readers to respond to error messages the come up (using `aria-live="assertive"` or `role="alert"` on error HTML elements). Currently with my static page, the screen reader doesn't pick up when an error occurs. With the static page, the user may be able to tab to the error but he would otherwise not know that his submission failed.
        - Character Count/Limit: I used jQuery to add `$(char).attr("role", "alert")` when user types more than 500 characters. When user goes under 500 characters, I use `removeAttr("role", "alert")` to remove the attribute.

      III) Implementing Aria attributes and best practices:
        1) WAIAble gem (https://github.com/techvision/waiable) -- This gem allows the developer a shortcut to adding standard aria attributes to forms. In short, it connects inputs to their labels, converts error messages into links (making them accessible via tab), adds the `aria-required="true"` to inputs when there is a model validation `presence: true`, and adds other notifiers when element attributes are present (such as announcing the size limitation when a `maxlength` is set to the input).
          Unfortunately, because of the way I structured my app, some of these features in the WAIAble gem are not that relevant, requiring me to insert aria attributes manually.
            (A) Because my textarea doesn't have a label, I need to add `aria-label` with a description, such as "Type your vent here." According to some style-guides, the `aria-label` property should be used when there is no label. For inputs with labels, use `aria-labelledby` and connect it to the id of the label.

            (B) Because my app's is mostly static and requires a refresh when submitting the form, screen readers won't be able to read my error messages unless the user tabs to it--this would likely cause the error to go unnoticed to visually impaired individuals. I would overcome this issue by submitting my form through AJAX and displaying errors with the `role="alert"` attribute via jQuery (see (D) below). Because nothing else on the page is moving, the screen reader will pick up the alert role attribute and read it back to the user right away. `aria-live="assertive"` does this as well.

            (C) The `aria-required="true"` addition that comes with the WAIAble gem is pretty useful. I just set the validation `presence: true` in the model, and the gem adds an `aria-required="true"` field to the textarea.

            (D) I would not set a maxlength attribute on my textarea because I don't want to limit the user from typing more than 500 characters; I just want to prevent him from submitting it. Instead, I would add in my `charCount` function 2 lines: `.attr("role", "alert")` when the character count exceeds 500; and `.removeAttr("role", "alert")` when the character count drops back below or equal to 500 characters.

        2) Aside from what WAIAble gives me, I would also add the following features to make my app more accessible:
          A) Features for a static page
            i) Set all my h2 elements to h1. The screen reader tells the user of the heading level, and having my main sections be called "heading level 2" makes it sound like a sub-section, which would be confusing.

            iii) `tabindex` - (https://www.paciellogroup.com/blog/2014/08/using-the-tabindex-attribute/)
              The tabindex attribute indicates that an element can be focused on, and determines how that focus is handled. A keyboard user will typically move through web content using the tab key, moving from one focusable element to the next in sequential order.
              - I decided to take the textarea autofocus off. I referred to Facebook and Twitter, and realized it is pretty standard for a non-accessible user to select the textarea before typing. This solves the problem of starting with focus in the middle of the page rather than at the top. Thus a visually impaired user might not be so confused when first landing on the page.

              - Because my page is structured in a weird order (the instructions block is set in reverse column), I have to specify the tabindexes for several elements, even ones that are already focusable. Since I want the instructions(right-block) to be accessed before the form, I set the nav and right-block to be `tabindex="1"` and set everything in the left-block (including the tab-able elements in the form) to be `tabindx="2"`.

              - Errors - `tabindex="-1"`
                Setting tabindex to "-1" makes the element focusable but outside the tab order--meaning the user cannot tab to the element but can click on it. This is useful for errors, which should be excluded from the tab order (https://www.paciellogroup.com/blog/2014/08/using-the-tabindex-attribute/). The idea behind this that the user should be notified of errors (otherwise he might think the form is broken and leave), but the errors should be outside the tab order so they don't interfere with the user navigating to the form (e.g., use JS/jQuery to show errors, and `aria-live="assertive"` to notify the user of these errors). Thus, I set my error list to `tabindex="-1"`.

        3) Other Considerations that were excluded:
          ii) `role` -- It seems to be standard practice to set a `role` to each tab-able element to serve as landmarks for the user. Landmarks help assistive technology users orient themselves to a page and helps them navigate easily to various sections of a page. These also provide an easy way for users to skip over blocks of content that are repeated on multiple pages and notify them of the programmatic structure of the page (https://www.w3.org/TR/WCAG20-TECHS/aria#ARIA11). So, I could do:
            - Nav => `role="navigation"`
            - Headers => `role="heading"`
            - Textarea => `role="textbox"`
            However, in HTML5, elements have default semantics. For example, nav already has `role="navigation"`, heading elements already have `role="heading"`, and textareas already have `role="textbox"`. Thus, applying these aria tags are redundant and unnecessary (https://w3c.github.io/aria-in-html/#aria-adds-nothing-to-default-semantics-of-most-html-elements). Using roles are more often used on the non-specific, div-like elements (http://stackoverflow.com/questions/22742066/aria-landmarks-not-working-in-textarea-input).
            - Errors => `role="alert"` -- The alert role is used to communicate an important and usually time-sensitive message to the user. When this role is added to an element, the browser will send out an accessible alert event to assistive technology products which can then notify the user about it. (https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Techniques/Using_the_alert_role)

____________________________________________________________________________________
____________________________________________________________________________________


<!--
    @What: WAI-ARIA, the Accessible Rich Internet Applications Suite, defines a way to make Web content and Web applications more accessible to people with disabilities. It especially helps with dynamic content and advanced user interface controls developed with Ajax, HTML, JavaScript, and related technologies. https://www.w3.org/WAI/intro/aria
      WAI-ARIA provides Web authors with the following:
        1. Roles to describe the type of widget presented, such as "menu", "treeitem", "slider", and "progressmeter"
        2. Roles to describe the structure of the Web page, such as headings, regions, and tables (grids)
        3. Properties to describe the state widgets are in, such as "checked" for a check box, or "haspopup" for a menu.
        4. Properties to define live regions of a page that are likely to get updates (such as stock quotes), as well as an interruption policy for those updates—for example, critical updates may be presented in an alert dialog box, and incidental updates occur within the page
        5. Properties for drag-and-drop that describe drag sources and drop targets
        6. A way to provide keyboard navigation for the Web objects and events, such as those mentioned above -->



  <!-- Static Features:
    `for` -- links labels to their inputs.
    `aria-labelledby` -- With the aria-labelledby attribute, authors can use a visible text element on the page as a label for a focusable element. (https://www.w3.org/TR/WCAG20-TECHS/aria#ARIA7)
    `aria-label` -- Same as aria-labelledby, but should be used when there is no text on the page that can be referenced to as the label. When a screen reader encounters the object, the aria-label text is read so that the user will know what it is.
    `aria-describedby` -- Using aria-describedby to provide descriptions of images when a short text alternative does not adequately convey the function or information provided in the object. (https://www.w3.org/TR/WCAG20-TECHS/aria#ARIA15)
    `role: "textbox"` -- The textbox role is used to identify an element that allows the input of free-form text. When this role is added to an element, the browser will send out an accessible textbox event to assistive technology products which can then notify the user about it.
      When the textbox role is added to an element, or such an element becomes visible, the user agent should do the following:
        1. Expose the element as having a textbox role in the operating system's accessibility API.
        2. Fire an accessible textbox event using the operating system's accessibility API if it supports it.
      Assistive technology products should listen for such an event and notify the user accordingly:
        1. Screen readers should announce its label and role when focus first lands on a textbox. If it also contains content, this should be announced as with a regular textbox.
        2. Screen magnifiers may enlarge the textbox.
          (https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Techniques/Using_the_textbox_role)
    `aria-multiline` -- use this because although inputs are usually submitted via ENTER or RETURN, textareas are not. This property informs the author of this distinction. Where this is multi-line and the input of line breaks is supported, e.g. when using a HTML <textarea>, it is necessary to also set aria-multiline=”true”.
      (https://www.w3.org/TR/wai-aria/states_and_properties#aria-multiline)
    `aria-required: "true"` -- property indicates that user input is required before(after?) submission. -->

  <!-- Dynamic Features:
    To change:
      1. `remote: "true"` -- this turns the request over to AJAX.

      2. `aria-hidden: "false"` --
        @What: [true] -- skips element/element is not identified by screen reader. [false] -- element is accessible.
        @Use: Toggle false for errors or anytime an element is shown/appears.

      3. `aria-live: "[assertive/passive]"` --
        @What: This tag indicates that an element will be updated, and describes the types of updates the user agents, assistive technologies, and user can expect from the live region.
        @Use: Notify when user exceeds 500 characters.
        @Other:
          (https://www.w3.org/TR/WCAG20-TECHS/aria#ARIA19)
            - The content within the aria-live region is automatically read by the AT (Assistive Technologies), without the AT having to focus on the place where the text is displayed.
            - `role=alert` which is equivalent to using `aria-live=assertive`.
            - Great Example Errors in jQuery! -->


      <!-- 4. `role: "alert"` --
        @What The alert role is used to communicate an important and usually time-sensitive message to the user. When this role is added to an element, the browser will send out an accessible alert event to assistive technology products which can then notify the user about it. (https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Techniques/Using_the_alert_role)
        @Use: Notify user of invalid submission if he submits a (1) blank comment, (2) comment less than 5 characters, (3) comment greater than 500 characters
        @Other:
          (https://www.w3.org/TR/WCAG20-TECHS/aria#ARIA4)
            - The objective of this technique is to define the role of an element using the role attribute with one of the non-abstract values defined in the WAI-ARIA Definition of Roles (https://www.w3.org/TR/wai-aria/roles#role_definitions). The WAI-ARIA specification provides an informative description of each role, how it relates to other roles, and the states and properties for each role.
          (https://www.w3.org/TR/WCAG20-TECHS/aria#ARIA11)
            - `Landmarks` -- The purpose of this technique is to provide programmatic access to sections of a web page. Landmark roles (or "landmarks") programmatically identify sections of a page. Landmarks help assistive technology (AT) users orient themselves to a page and help them navigate easily to various sections of a page.
            - They also provide an easy way for users of assistive technology to skip over blocks of content that are repeated on multiple pages and notify them of programmatic structure of a page. For instance, if there is a common navigation menu found on every page, landmark roles (or "landmarks") can be used to skip over it and navigate from section to section. This will save assistive technology users and keyboard users the trouble and time of tabbing through a large amount of content to find what they are really after, much like a traditional "skip links" mechanism. -->

<!--
  jQuery:
    jQuery attr() Method -- The attr() method sets or returns attributes and values of the selected elements.
      (https://www.w3schools.com/jquery/html_attr.asp)
      (http://stackoverflow.com/questions/27752210/how-to-use-jquery-how-to-change-the-aria-expanded-false-part-of-a-dom-element)

    Toggle Char max exceeded -- when char > 500, toggle class ON to notify that char limit exceeded. When <= 500, toggle class OFF. -->





Good source!
  https://www.w3.org/TR/WCAG20-TECHS/aria



  @INTERNATIONALIZATION : Along similar lines, how much work would be involved to adapt this app for internationalization?






  @RULES : Logic that holds the "rules" of all text entries must be between 5 and 500 characters could be abstracted and made more explicit. Right now its spread out to a lot of places.

  - There is a comments folder with a form in it?
