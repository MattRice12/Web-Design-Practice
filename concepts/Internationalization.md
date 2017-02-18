https://developer.apple.com/library/content/documentation/MacOSX/Conceptual/BPInternational/Introduction/Introduction.html

I18n gem:
  https://github.com/svenfuchs/i18n
  http://guides.rubyonrails.org/i18n.html



  ____________________________________________________________________________________
  ____________________________________________________________________________________


@INTERNATIONALIZATION : Along similar lines, how much work would be involved to adapt this app for internationalization?

RailsGuides offers a very thorough guide for internationalizing an app (
http://guides.rubyonrails.org/i18n.html). I referred to it primarily when implementing the i18n gem. I used stackoverflow and a few other sources to fill in the gaps for more specific issues.

Internationalization is the process of making your app able to adapt to different languages, regions, and cultures. An internationalized app appears as if it is a native app in all the languages and regions it supports.

i18n gem -- The Ruby I18n gem which is shipped with Ruby on Rails provides an easy-to-use and extensible framework for translating your application to a single custom language other than English or for providing multi-language support in your application.

In the process of internationalizing your Rails application you have to:
  - Ensure you have support for i18n.
  - Tell Rails where to find locale dictionaries.
  - Tell Rails how to set, preserve and switch locales.

In the process of localizing your application you'll probably want to do the following three things:
  - Replace or supplement Rails' default locale - e.g. date and time formats, month names, Active Record model names, etc.
  - Abstract strings in your application into keyed dictionaries - e.g. flash messages, static text in your views, etc.
  - Store the resulting dictionaries somewhere.

____________________________________________________________________________________

I. Overview:
  First, I need to internationalize my application by abstracting every locale-specific element.
    - Manually -- I could let the user manually select his locale (in the nav, give the user the options "en, es, fr, [etc.]"). However, this is a very clunky way to do it and puts an unnecessary burden on the user. To make this less clunky, I could have a `settings` page where the user can select his preferred language.
    - Automatically -- My app could detect user's language based on the language settings in the user's web browser. I prefer this method because it is much cleaner and automatic. The downside is that this restricts the user's choice; however, it isn't much of a downside.

  Second, I need to localize my application by providing necessary translations for these abstracts.
    - Localizing involves (1) defining the dictionaries for each language you support in `config/locales/*.yml`, and (2) standardizing the views to access the correct dictionary associated with the user's language.

II. Internationalize:
  A. Ensure you have support for i18n.
    - i18n has shipped with Rails since Rails 2.2

  B. Tell Rails where to find locale dictionaries.
    - Rails adds all .rb and .yml files from the config/locales directory to the translations load path, automatically.

  C. Tell Rails how to set, preserve and switch locales. The default locale (en) is used for all translations unless I18n.locale is explicitly set.
    1. Manual -- User can selects locale
      *Routes*
        Set up your routes to insert the locale into the URL: `www.venter.com/en/vents`. Note the root is outside the locale scope; this sets the root as having the default locale of `en`. And rightly so--there should be only one root.
        ```
          scope "(:locale)", locale: /en|es|fr/ do
            resources :vents
          end
          root 'vents#index'
        ```

      *application_controller.rb*
        The applications controller detects the locale by looking in the params and sets it to be used in the views (see III.B. below).
        ```
          before_action :set_locale

          def set_locale
            I18n.locale = params[:locale] || I18n.default_locale
          end
        ```
      *_nav.html.erb*
        Set up a selection for the user to choose his locale. A common place to put this is in the nav; however, putting it in a settings menu also makes sense.
        ```
          <div class="languages">
            Languages:
            <%= link_to_unless_current "English", locale: "en" %> |
            <%= link_to_unless_current "Spanish", locale: "es" %> |
            <%= link_to_unless_current "French", locale: "fr" %>
          </div>
        ```

    2. Automatic - Detect locale automatically based on Language Header
        *gem 'http_accept_language'*
          https://github.com/iain/http_accept_language
          I (and RailsGuides) recommend using this. It detects the users preferred language, as sent by the "Accept-Language" HTTP header. To change your preferred language, you will have to do so in your browser settings rather than in the app.

        *Routes*
          Return routes to normal. Since the gem detects the user's preferred language through the browser, we don't need to set a scope for the locale.
          ```
            resources :vents
            root 'vents#index'
          ```

        *application_controller.rb*
          Again, the application controller sets the locale. Here, we set it according to what locale the http_accept_language gem detects.
          ```
            include HttpAcceptLanguage::AutoLocale

            before_action :set_locale

            def set_locale
              I18n.locale = http_accept_language.compatible_language_from(I18n.available_locales)
            end
        ```

        *config/application.rb*
          Set your available locales here. In my app, I'm only translating English, Spanish, and French (for now).
          `config.i18n.available_locales = %w(en es fr)`

III) Localize:
  A. Replace or supplement Rails' default locale - e.g. date and time formats, month   names, Active Record model names, etc.
    I did not need to do this step since my times are all relative rather than absolute (e.g., Did not need to differentiate between 11/2/2017 and 2/11/2017). However, I did add the datetime libraries in part C below.

  B. Abstract strings in your application into keyed dictionaries - e.g. flash messages, static text in your views, etc.
    1. ERB -- Abstract strings in the views by calling the correct branch in your .yml files (set up in part C). Call them by putting t("[abstraction]") between two erb tags.
      *_.html.erb*
        `<h1><%= t("vent.welcome")%></h1>`

    2. JavaScript (http://stackoverflow.com/questions/2701749/rails-internationalization-of-javascript-strings)
      JavaScript requires a few more steps:
      *application.html.erb*
        First, you have to define the t method.
        ```
          <script type="text/javascript">
            window.t = <%= current_translations.to_json.html_safe %>
          </script>
        ```

      *application_helper.rb*
        ```
          def current_translations
            @translations ||= I18n.backend.send(:translations)
            @translations[I18n.locale].with_indifferent_access
          end
        ```

      *application.js*
        Only then can you use the t method. It looks a little different than you would type in ERB, but it works the same way.
        ```
          $("form").on("ajax:success",function(e, data, status, xhr){
            $('#vent-index').prepend("<li class='vent'><p>" + data.text + "</p><sub>" + t.time.submitted.one + "</sub></li>");
          });
        ```

  C. Store the resulting dictionaries somewhere.
    When we call the t method in part B of this section, it searches the dictionaries we make for the appropriate translation. So if my locale is set to `en` and I translate `<%= t("vent.welcome") %>`, i18n will search `config/locals/*.yml` until it finds:
    *config/locales/en.yml*
      ```
        en:
          vent:
            welcome: "Welcome to Venter"
      ```

    *config/locales/es.yml*
    If my local is set to `es`, i18n will search for:
      ```
        es:
          vent:
            welcome: "Bienvenido a Venter"
      ```

    *Datetimes and Helper Methods*
      - This was the last and probably the most difficult part of localizing my app. It involved getting the following helper method to be translated between languages:
        ```
          def submit_time(vent)
            t("time.submitted", count: distance_of_time_in_words(Time.now, vent.created_at))
          end
        ```

      In my `config/locales/es.yml` file, I had:
        ```
          es:
            time:
              submitted: "Submitted %{count} ago."
        ```

      - However, this was not enough to translate my helper method. The reason is that the output was a *datetime* type rather than a string type. Thus, it required more work to translate.

      - When localizing datetimes, you need to import the library. So, I copied the content I wanted (datetimes) from https://github.com/svenfuchs/rails-i18n/tree/master/rails/locale, and pasted them in the yaml file (You don't need to do this for English, since it comes stock; but I'm using `en` rather than `es` or a different language for explanation. See below for an `es` example.):
      ```
        en:
          time:
            submitted: "Submitted %{count} ago."

          datetime:
            distance_in_words:
              about_x_hours:
                one: about 1 hour
                other: about %{count} hours
      ```

      - Even though I imported `datetime: ...`, I'm not calling it specifically in my helper method. I'm still calling `t(time.submitted, count: distance_of...)`. The purpose of importing `datetime` is just for i18n to use in its own translation of any datetimes it comes across. Likely, you will want to customize the output, especially since you won't be certain whether you'll need to use `about_x_hours` or `less_than_x_minutes` or whatever else.

      - Both `datetime:` and `time: submitted: "..."` work together in translating the output. First, i18n looks to the `about_x_hours` branch to output the translation into `about [n] hours`; then it looks to the `submitted` branch to output the translation of `Submitted [about_x_hours] ago.` So ultimately we get something like `Submitted about 3 hours ago.`

      - Does that make sense? Yes. But it wasn't super intuitive. It really took a lot of playing around to figure out how it worked. In the end, my es.yml file looks something like this (though, in my actual file, I included all the datetime translations, just for good measure):
        ```
          es:
            time:
              submitted:
                one: Enviado ahora.
                other: "Alrededor de %{count}"

            datetime:
              distance_in_words:
                about_x_hours:
                  one: Alrededor de 1 hora
                  other: "%{count} horas"
                x_minutes:
                  one: 1 minuto
                  other: "%{count} minutos"
                ...
        ```







    ____________________________________________________________________________________
    ____________________________________________________________________________________



    @RULES : Logic that holds the "rules" of all text entries must be between 5 and 500 characters could be abstracted and made more explicit. Right now its spread out to a lot of places.

    - There is a comments folder with a form in it?
