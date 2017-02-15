I do have a few pieces of feedback and a few notes/questions that I wrote down while reading through your submission. Some of these questions are open ended and may not require an answer, but if you have an answer, I'd be happy to hear it.

  @ACCESSIBILITY : How would you go about making this application more accessible those with physical disabilities or who are visually impaired? What changes would need to be made and what are the best practices in Rails and Jquery for using Aria attributes?
    @What: WAI-ARIA, the Accessible Rich Internet Applications Suite, defines a way to make Web content and Web applications more accessible to people with disabilities. It especially helps with dynamic content and advanced user interface controls developed with Ajax, HTML, JavaScript, and related technologies. https://www.w3.org/WAI/intro/aria
      WAI-ARIA provides Web authors with the following:
        1. Roles to describe the type of widget presented, such as "menu", "treeitem", "slider", and "progressmeter"
        2. Roles to describe the structure of the Web page, such as headings, regions, and tables (grids)
        3. Properties to describe the state widgets are in, such as "checked" for a check box, or "haspopup" for a menu.
        4. Properties to define live regions of a page that are likely to get updates (such as stock quotes), as well as an interruption policy for those updates—for example, critical updates may be presented in an alert dialog box, and incidental updates occur within the page
        5. Properties for drag-and-drop that describe drag sources and drop targets
        6. A way to provide keyboard navigation for the Web objects and events, such as those mentioned above


    I looked into it a bit and found a Rails gem for it--WAIAble (https://github.com/techvision/waiable). The gem makes rails forms more accessible for screen readers and keyboard users. This gem seems to respond based the presence of elements and properties:

  Static Features:
    `for` -- links labels to their inputs.
    `aria-labelledby` / `aria-describedby` -- Identifies the element (or elements) that labels the current element.
    `aria-label` -- Same as aria-labelledby, but should be used when the label is not visible on the screen.
    `role: "textbox"` -- The textbox role is used to identify an element that allows the input of free-form text. When this role is added to an element, the browser will send out an accessible textbox event to assistive technology products which can then notify the user about it.
      When the textbox role is added to an element, or such an element becomes visible, the user agent should do the following:
        1. Expose the element as having a textbox role in the operating system's accessibility API.
        2. Fire an accessible textbox event using the operating system's accessibility API if it supports it.
      Assistive technology products should listen for such an event and notify the user accordingly:
        1. Screen readers should announce its label and role when focus first lands on a textbox. If it also contains content, this should be announced as with a regular textbox.
        2. Screen magnifiers may enlarge the textbox.
          (https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Techniques/Using_the_textbox_role)
    `aria-multiline` -- use this because although inputs are usually submitted via ENTER or RETURN, textareas are not. This property informs the author of this distinction.
      (https://www.w3.org/TR/wai-aria/states_and_properties#aria-multiline)

  Dynamic Features:
    To change:
      1. `remote: "true"` -- this turns the request over to AJAX.

      2. `aria-hidden: "false"` --
        @What: [true] -- skips element/element is not identified by screen reader. [false] -- element is accessible.
        @Use: Toggle false for errors or anytime an element is shown/appears.

      3. `aria-live: "[assertive/passive]"` --
        @What: This tag indicates that an element will be updated, and describes the types of updates the user agents, assistive technologies, and user can expect from the live region.
        @Use: Notify when user exceeds 500 characters.

      4. `role: "alert"` --
        @What The alert role is used to communicate an important and usually time-sensitive message to the user. When this role is added to an element, the browser will send out an accessible alert event to assistive technology products which can then notify the user about it. (https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Techniques/Using_the_alert_role)
        @Use: Notify user of invalid submission if he submits a (1) blank comment, (2) comment less than 5 characters, (3) comment greater than 500 characters



  jQuery:
    jQuery attr() Method -- The attr() method sets or returns attributes and values of the selected elements.
      (https://www.w3schools.com/jquery/html_attr.asp)
      (http://stackoverflow.com/questions/27752210/how-to-use-jquery-how-to-change-the-aria-expanded-false-part-of-a-dom-element)


    Additionally, since I'm minimizing the clutter on the screen and since I only have a single input field, I prefer to excluding labels. Instead, I added the `aria-label` property to the textarea and included the information I want it to speak: `"aria-label": "Input Text, Must be between 5 and 500 characters."`, or something similar.
      (https://www.w3.org/TR/wai-aria/states_and_properties#aria-label)

    The shortcoming for this is if the user submits a comment that falls outside the 5-500 char count parameters--the

    First, I found that adding the `aria-label` property to the textarea worked better than creating a label element with the `for` tag.  (http://www.deque.com/blog/accessible-client-side-form-validation-html5/).

    Second, the gem adds an aria-describedby property to announce the size limitation when using the `maxlength` property. I find this useful, but not in my current app, since in addition to announcing the size limitation, the maxlength property also prevents the user from exceeding the size limitation. Rather, I want the user to be able to type as much as he wants and is only prevent from submitting when the maximum character limit is exceeded (like twitter). Additionally, WAIABle only announces

    Third, although WAIAble does provide feedback on error messages, it seems to do so only in certain situations.


when using a HTML <textarea>, it is necessary to also set aria-multiline=”true”.

    Second, the user needs to be notified when he exceeds the max character length. This is a bit trickier since I am deliberately trying to not restrict the user from typing more than 500 characters. Rather, I'm only trying to prevent the user from submitting a post if it is longer than 500 characters (like twitter). To solve this, I think I would use JavaScript to trigger (1) when the user first exceeds 500 characters, and (2) if the user leaves the textarea when the textarea has > 500 characters (http://stackoverflow.com/questions/24036568/how-to-make-maxlength-attribute-accessible). This is a little extra work, but



    For text field and text area, when maxlength option is provided, aria-describedby property is added to that form field which will announce about the size limitation of that field to the screen reader. On form submission, if there is any error then these error messages are linkable and clicking on that link will shift the focus to the form field for which the error gets generated. Additionally the error message id will be added to the aria-describedby property which will announce the error message to the screen reader when the focus is on errorneous input field.






  @INTERNATIONALIZATION : Along similar lines, how much work would be involved to adapt this app for internationalization?






  @RULES : Logic that holds the "rules" of all text entries must be between 5 and 500 characters could be abstracted and made more explicit. Right now its spread out to a lot of places.

  - There is a comments folder with a form in it?
