  @ACCESSIBILITY : How would you go about making this application more accessible those with physical disabilities or who are visually impaired? What changes would need to be made and what are the best practices in Rails and Jquery for using Aria attributes?
    I) Introduction -- To put it briefly, I would convert my app into something more dynamic so that I could take advantage of the `role="alert"` attribute to announce specific feedback related to the user's submission (whether the submission is successful or erroneous). Additionally, I would set tabindexes on several of the main elements (nav, headers, textarea) so that these main elements serve as landmarks and the page is easier to navigate for the user. Finally, I would add an `aria-label` attribute to the textarea, briefly describing what it is and the character limit. The default semantics for most of these elements take care of the rest.

    II) Aria
      1) What it is:
        Aria attributes make content more navigable and understandable. Aria attributes are added to HTML elements so that screen readers can pick them up and keyboard users can navigate to these elements. It especially helps with dynamic content and advanced user interface controls developed with Ajax, HTML, JavaScript, and related technologies." (https://www.w3.org/WAI/intro/aria)

      2) Aria attributes and best practices:
        i) `role` - (https://www.w3.org/TR/WCAG20-TECHS/aria#ARIA11)
          Add `role` to each tab-able element to serve as landmarks for the user. Landmarks help assistive technology users orient themselves to a page and helps them navigate easily to various sections of a page. These also provide an easy way for users to skip over blocks of content that are repeated on multiple pages and notify them of the programmatic structure of the page. So, I could do:
            - Nav => `role="navigation"`
            - Headers => `role="heading"`
            - Textarea => `role="textbox"`
              However, in HTML5, these elements have default semantics. For example, nav already has `role="navigation"`, heading elements already have `role="heading"`, and textareas already have `role="textbox"`. Thus, applying these aria tags are redundant and unnecessary (https://w3c.github.io/aria-in-html/#aria-adds-nothing-to-default-semantics-of-most-html-elements). Using roles are more often used on the non-specific, div-like elements (http://stackoverflow.com/questions/22742066/aria-landmarks-not-working-in-textarea-input).
            - Success/Errors => `role="alert"` -- The alert role is used to communicate an important and usually time-sensitive message to the user. When this role is added to an element, the browser will send out an accessible alert event to assistive technology products which can then notify the user about it. (https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Techniques/Using_the_alert_role)

        ii) `aria-labelledby/describedby/label` - (https://www.w3.org/TR/wai-aria/states_and_properties)
          All form fields should have labels or some way for a screen reader to inform the user what the input field represents. Typically the developer would use `aria-labelledby` to connect the input to the label via the label's id. `aria-describedby` is used for longer descriptions. `aria-label` is used when an input field has no associated label--e.g., the textarea in my app. However like `role`, none of these (except maybe are necessary since the default semantics of input and label elements already add this description. These would be used in situations where you give custom descriptions or for some reason convert neutral elements (like a div) into an input field.

        iii) `tabindex` - (https://www.paciellogroup.com/blog/2014/08/using-the-tabindex-attribute/)
          The tabindex attribute indicates that an element can be focused on, and determines how that focus is handled. A keyboard user will typically move through web content using the tab key, moving from one focusable element to the next in sequential order.
            - `tabindex="0"` -- When tabindex is set to 0, the element is inserted into the tab order based on its location in the source code.
            - `tabindex="+n"` -- A tabindex of 1 or greater imposes a custom tab order (lower numbers are tabbed to first).
            - `tabindex="-n"` -- Setting tabindex to "-1" makes the element focusable but outside the tab order. This is useful for errors, which should be excluded from the tab order. The idea behind this that the user should be notified of errors (otherwise he might think the form is broken and leave), but the errors should be outside the tab order so they don't interfere with the user navigating to the form (e.g., use JS/jQuery to show errors, and `role="alert"` or `aria-live="assertive"` to notify the user of these errors).

    III) My Changes
      1) jQuery:
        i) Errors & Submission: I would use jQuery in combination with Aria attributes to help accessibility. I would use jQuery and AJAX to handle form submission, making my app more dynamic. The benefit of doing so would be for screen readers to pick up success and error messages that appear (using `aria-live="assertive"` or `role="alert"` on error HTML elements). Currently with my static page, the screen reader won't recognize an error message pops up. This is because the page refreshes with the success/error message. While the user is still able to tab to the success/error message, he will not be immediately aware of the outcome, and may assume the app is broken.

        ii) Character Count/Limit: I used jQuery to add `$(char).attr("role", "alert")` when user types more than 500 characters. When the character count equals or drops below 500 characters, I use `removeAttr("role", "alert")` to remove the attribute.

      2) Aria:
        i) `textarea` -- To label my textarea, I would add the `aria-label` attribute with a custom description, such as "Type your vent here."

        ii) `heading elements` -- Set all my h2 elements to h1. The screen reader tells the user of the heading level, and having my main sections be called "heading level 2" makes it sound like a sub-section, which would be confusing.

        iii) `textarea autofocus` -- I decided to take the textarea autofocus off. I referred to Facebook and Twitter, and realized it is pretty standard for a non-accessible user to select the textarea before typing. This solves the problem of starting with focus in the middle of the page rather than at the top. Thus a visually impaired user might not be so confused when first landing on the page.

        iv) `tabindex` -- Because my page is structured in a weird order (the .content class has `flex-wrap: wrap-reverse;`), I need to specify the tabindexes for several elements, even ones that are already focusable. Since I want the instructions(.right-block) to be accessed before the form, I set the nav and .right-block to be `tabindex="1"` and set everything in the left-block (including the tab-able elements in the form) to be `tabindx="2"`.
        I also set my error list to `tabindex="-1"` and add `role="alert"` so that the screen reader picks them up but they don't interrupt the tab order

    IV) WAIAble gem -- (https://github.com/techvision/waiable)
      This gem allows the developer a shortcut to adding standard aria attributes to forms. In short, it connects inputs to their labels, converts error messages into links (making them accessible via tab), adds the `aria-required="true"` to inputs when there is a model validation `presence: true`, and adds other notifiers when element attributes are present (such as announcing the size limitation when a `maxlength` is set to the input).
      Unfortunately, because of the way I structured my app, some of these features in the WAIAble gem are not that relevant. Additionally, at this point it seems easier to just implement them manually.
        i) Because my textarea doesn't have a label, WAIAble won't add the `aria-labelledby` attribute to my textarea.

        ii) Because my app is mostly static and requires a refresh when submitting the form, screen readers won't be able to read my error messages unless the user tabs to it. And if I switch to a more dynamic app, I want my errors to have a tabindex of "-1", which makes WAIAble converting them into links obsolete.

        iii) The `aria-required="true"` addition that comes with the WAIAble gem is pretty useful. I just set the validation `presence: true` in the model, and the gem adds an `aria-required="true"` field to the textarea.

        iv) I would not set a maxlength attribute on my textarea because I don't want to limit the user from typing more than 500 characters; I just want to prevent him from submitting it.
