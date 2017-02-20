@ACCESSIBILITY : How would you go about making this application more accessible those with physical disabilities or who are visually impaired? What changes would need to be made and what are the best practices in Rails and Jquery for using Aria attributes?

  As part of a code challenge, I was tasked with determining how to make an app accessible to users with physical or visual disabilities. I was to explain the best practices in Rails and jQuery using Aria attributes (the app I was making accessible was a Rails app using jQuery). First, I will explain what Aria attributes are and many of the best practices for implementing them into a simple app. Then, I will discuss how I would go about implementing Aria attributes into my app, which is a simple app with a textarea and output section.

  I) Aria
    1) What it is:
      Aria attributes make content more navigable and understandable. Aria attributes are added to HTML elements so that screen readers can pick them up and keyboard users can navigate to these elements. It especially helps with dynamic content and advanced user interface controls developed with Ajax, HTML, JavaScript, and related technologies. (https://www.w3.org/WAI/intro/aria)

    2) Aria attributes and best practices:
      i) Default Semantics --
        Before going into specific attributes, it is important to note than in HTML5 most elements have default semantics that are exposed to the browser and are read by screen readers. Default semantics just describe what the element is. For example, an h1 has a `heading` role with a `level 1` status. As a result, I do not need to manually set the h1's role or level. Doing so is redundant and generally not advisable.
          (https://w3c.github.io/aria-in-html/#aria-adds-nothing-to-default-semantics-of-most-html-elements)
          (https://www.sitepoint.com/avoiding-redundancy-wai-aria-html-pages/)

        A benefit of writing good HTML is that you get a good amount of accessibility for free.

        - Contrarily, some elements like the `div` do not have such default semantics, and if I want to make it a `heading`, then I need to give it the `role="heading"` attribute. Using roles are more often used on the non-specific, div-like elements
          (http://stackoverflow.com/questions/22742066/aria-landmarks-not-working-in-textarea-input)
          http://vanseodesign.com/web-design/five-rules-aria-html/

      i) `role` - (https://www.w3.org/TR/WCAG20-TECHS/aria#ARIA11)
        So what are roles anyways and why do we need them? Roles allow elements to serve as landmarks for the user. Landmarks help assistive technology users orient themselves to a page and helps them navigate easily to various sections of a page. As a user tabs through the page, the screen reader will announce the element's role. This way, the user knows whether he or she tabbed to an element with a `navigation` role, a `main` role, a `region` role, a `form` role, or a different role. Here is a list of roles (https://www.w3.org/TR/wai-aria/roles#roles_categorization).

        - Success/Errors => `role="alert"` -- The alert role is used to communicate an important and usually time-sensitive message to the user. When this role is added to an element, the browser will send out an accessible alert event to assistive technology products which can then notify the user about it. (https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Techniques/Using_the_alert_role)

      ii) `aria-live` -- An alternative to setting an element's role to `alert` is to set the region to `aria-live="assertive"`. Both alert and aria-live work with a dynamic webpage/application and announce when that content changes. Aria-live has two settings "polite" and "assertive" -- polite waits until the screen reader is finished with its current announcements; assertive interrupts its current announcements and gets announced immediately.

      iii) `aria-labelledby/describedby/label` - (https://www.w3.org/TR/wai-aria/states_and_properties)
        All form fields should have labels or some way for a screen reader to inform the user what the input field represents. Typically the developer would use `aria-labelledby` to connect the input to the label via the label's id. `aria-describedby` is used for longer descriptions. `aria-label` is used when an input field has no associated label--e.g., the textarea in my app.

      iv) `tabindex` - (https://www.paciellogroup.com/blog/2014/08/using-the-tabindex-attribute/)
        The tabindex attribute indicates that an element can be focused on, and determines how that focus is handled. A keyboard user will typically move through web content using the tab key, moving from one focusable element to the next in sequential order.
          - `tabindex="0"` -- When tabindex is set to 0, the element is inserted into the tab order based on its location in the source code.
          - `tabindex="+n"` -- A tabindex of 1 or greater imposes a custom tab order (lower numbers are tabbed to first).
          - `tabindex="-n"` -- Setting tabindex to "-1" makes the element focusable but outside the tab order. This is useful for errors, which should be excluded from the tab order. The idea behind this that the user should be notified of errors (otherwise he might think the form is broken and leave), but the errors should be outside the tab order so they don't interfere with the user navigating to the form (e.g., use JS/jQuery to show errors, and `role="alert"` or `aria-live="assertive"` to notify the user of these errors).

  II) My Changes
    1) Overview:
      To make my app accessible, I would do the following: First, I would make my app more dynamic so that I could take advantage of the `role="alert"` attribute to announce whether the submission was successful or erroneous. Second, I would set tabindexes on several of the main elements (nav, headers, textarea) so that these main elements serve as landmarks and the page is easier to navigate for the user. Finally, I would add an `aria-label` attribute to the textarea with a brief of its purpose and its character limit, and I would change my heading elements to h1. I wouldn't need to add any other roles since the default semantics for most of these elements take care of that. I discuss the WAIAble gem last, but I end up not using it because it doesn't offer me anything that I can't do easily (though it might be useful in larger apps).

    2) jQuery:
      i) Live Regions / Alert: To notify the user of erroneous or successful submissions, I would first make the content more dynamic, by using jQuery and AJAX for submission. Then I would set the element containing success/error messages to be live: `aria-live="assertive"`; alternatively, I could alert the user when there is a change: `role="alert"`. Currently with my static page, the screen reader doesn't recognize my error messages when they pop up (even with the WAIAble gem, the screen reader will only read the error in certain, limited situations).

      ii) Character Count/Limit: I used jQuery to add `$(char).attr("role", "alert")` when user types more than 500 characters. When the character count equals or drops below 500 characters, I use `removeAttr("role", "alert")` to remove the attribute.

    3) Aria:
      i) `textarea` -- To label my textarea, I would add the `aria-label` attribute with a custom description, such as "Type your vent here. Vents must be between 5 and 500 characters."

      ii) `heading elements / aria-level` -- Set all my h2 elements to h1. The screen reader tells the user of the heading level, and having my main sections be called "heading level 2" makes it sound like a sub-section, which would be confusing. Alternatively, I could keep my headings at h2 for styling purposes and set `aria-level="1"`. However, there really is no reason to prefer h2 over h1 for styling, so I'm better off changing the HTML element to h1 and using its implicit semantics to announce the heading level.

      iii) `textarea autofocus` -- I decided to take the textarea autofocus off. I referred to Facebook and Twitter, and realized it is pretty standard for a non-accessible user to select the textarea before typing. This solves the problem of starting with focus in the middle of the page rather than at the top. Thus a visually impaired user might not be so confused when first landing on the page.

      iv) `tabindex` -- Because my page is structured in a weird order (the .content class has `flex-wrap: wrap-reverse;`), I need to specify the tabindexes for several elements, even ones that are already focusable. Since I want the instructions(.right-block) to be accessed before the form, I set the nav and .right-block to be `tabindex="1"` and set everything in the left-block (including the tab-able elements in the form) to be `tabindx="2"`.
      I also set my error list to `tabindex="-1"` and add `role="alert"` so that the screen reader picks them up but they don't interrupt the tab order

      v) `roles` -- As mentioned in section I, roles are often redundant when default semantics already gives the element a role. In my app, every element that would serve as a landmark already have default semantics defining their roles. So, I would not need to specify any roles (except when using `role="alert"` for successful/erroneous submissions)

    4) WAIAble gem -- (https://github.com/techvision/waiable)
      This gem allows the developer a shortcut to adding standard aria attributes to forms. In short, it connects inputs to their labels, converts error messages into links (making them accessible via tab), adds the `aria-required="true"` to inputs when there is a model validation `presence: true`, and adds other notifiers when element attributes are present (such as announcing the size limitation when a `maxlength` is set to the input).
      Unfortunately, because of the way I structured my app, some of these features in the WAIAble gem are not that relevant. Additionally, at this point it seems easier to just implement them manually.
        i) Because my textarea doesn't have a label, WAIAble won't add the `aria-labelledby` attribute to my textarea.

        ii) Because my app is mostly static and requires a refresh when submitting the form, screen readers won't be able to read my error messages unless the user tabs to it. And if I switch to a more dynamic app, I want my errors to have a tabindex of "-1", which makes WAIAble converting them into links obsolete.

        iii) The `aria-required="true"` addition that comes with the WAIAble gem is pretty useful. I just set the validation `presence: true` in the model, and the gem adds an `aria-required="true"` field to the textarea. But I could just as easily put `aria-required="true"` in my HTML.

        iv) I would not set a maxlength attribute on my textarea because I don't want to limit the user from typing more than 500 characters; I just want to prevent him from submitting it.
