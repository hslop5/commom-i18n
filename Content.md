Start common libs i18n idea

#1 Tạo utils để chứa thông tin basic app support ngôn ngữ nào
```
public class LocalesUtils {
    private static final List<Locale> LOCALES = Arrays.asList(
            AppConstants.LOCALE_VI,
            AppConstants.LOCALE_EN,
            AppConstants.LOCALE_VN
    );

    public static Locale resolveLocale(String headerLang) {
        if (StringUtils.isEmpty(headerLang)) {
            return Locale.getDefault();
        }
        List<Locale.LanguageRange> list = Locale.LanguageRange.parse(headerLang);
        return Locale.lookup(list, LOCALES);
    }

    public static List<Locale> getLocales() {
        return LOCALES;
    }

    public static Locale resolveLocale() {
        // Use Spring provided API
        Locale locale = null;
        try {
            HttpServletRequest servletRequest = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
            String headerLang = servletRequest.getHeader(HttpHeaders.ACCEPT_LANGUAGE);
            if (StringUtils.isNotEmpty(headerLang)) {
                locale = LocalesUtils.resolveLocale(headerLang);
            }
        } catch (Exception ignored) {
        }
        if (null == locale) {
            locale = LocaleContextHolder.getLocale();
        }

        return locale;
    }
}
```

#1 Tạo BasicMessageSourceConfigurer
trong app sẽ extend class này với @Configurer

```
public class BasicMessageSourceConfigurer extends AcceptHeaderLocaleResolver implements WebMvcConfigurer {

    @Override
    public Locale resolveLocale(HttpServletRequest request) {
        String headerLang = request.getHeader("Accept-Language");
        LocaleContextHolder.setDefaultLocale(AppConstants.LOCALE_VI);
        LocaleContextHolder.setLocale(AppConstants.LOCALE_VI);

        return headerLang == null || headerLang.isBlank()
                ? AppConstants.LOCALE_VI
                : Locale.lookup(Locale.LanguageRange.parse(headerLang), LocalesUtils.getLocales());
    }

    @Bean
    LocaleResolver localeResolver() {
        AcceptHeaderLocaleResolver localeResolver = new AcceptHeaderLocaleResolver();
        localeResolver.setDefaultLocale(AppConstants.LOCALE_VI);
        return localeResolver;
    }

    @Bean
    public ResourceBundleMessageSource messageSource() {
        ResourceBundleMessageSource rs = new ResourceBundleMessageSource();
        rs.setBasenames(getMessageSourceBasenames());
        rs.setDefaultEncoding(StandardCharsets.UTF_8.name());
        rs.setUseCodeAsDefaultMessage(true);
        return rs;
    }

    protected String[] getMessageSourceBasenames() {
        return new String[]{"i18n/message"};
    }
}
```
#1 Dùng ControlAdvice kết hợp với Customer Exception để handler parameter 
