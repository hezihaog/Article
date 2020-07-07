#### SpringMVC解决跨域问题

跨域一般出现在前端调用后端接口上，当前端发起Ajax请求，请求地址和当前前端网站不同域名、不同端口时，就会出现跨域问题，这是浏览器出于安全考虑做出的限制。

#### 全局配置

要注意以下几个地方：

- addAllowedOrigin，添加允许的域（可以添加多个），千万不能写*，否则否则cookie就无法使用了
- addAllowedMethod，添加请求方式，一般常用就是GET和POST，你也可以添加其他的，例如HEAD、PUT等，这里，我都添加了
- addAllowedHeader，设置必须传递的请求头，你可以在这里做限定，带了指定请求头的请求才允许通过
- registerCorsConfiguration，添加映射路径，一般我们都是拦截一切请求

除了添加允许的域，其他基本复制粘贴就能用啦。

```
/**
 * 跨域配置
 */
@Configuration
public class GlobalCorsConfig {
    @Bean
    public CorsFilter corsFilter() {
        //1.添加CORS配置信息
        CorsConfiguration config = new CorsConfiguration();
        //1) 允许的域,不要写*，否则cookie就无法使用了
        config.addAllowedOrigin("http://manage.leyou.com");
        //2) 是否发送Cookie信息
        config.setAllowCredentials(true);
        //3) 允许的请求方式
        config.addAllowedMethod("OPTIONS");
        config.addAllowedMethod("HEAD");
        config.addAllowedMethod("GET");
        config.addAllowedMethod("PUT");
        config.addAllowedMethod("POST");
        config.addAllowedMethod("DELETE");
        config.addAllowedMethod("PATCH");
        //4）允许的头信息
        config.addAllowedHeader("*");
        //3）预检请求的有效时长，超过后，就会先发一次预检请求来验证是否允许
        config.setMaxAge(3600L);//1个小时

        //2.添加映射路径，我们拦截一切请求
        UrlBasedCorsConfigurationSource configSource = new UrlBasedCorsConfigurationSource();
        configSource.registerCorsConfiguration("/**", config);

        //3.返回新的CorsFilter.
        return new CorsFilter(configSource);
    }
}
```