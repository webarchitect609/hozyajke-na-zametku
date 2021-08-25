HttpException внутри ParamConverter
===================================

###### 2021-02-11

Почему-то явно нигде не говорится, что в Symfony в конвертере
типа `\Sensio\Bundle\FrameworkExtraBundle\Request\ParamConverter\ParamConverterInterface` можно кидать исключения
типа `\Symfony\Component\HttpKernel\Exception\HttpExceptionInterface` и они корректно отработают. Т.е. в конверторе
можно и нужно заниматься валидацией параметров запроса, требуемых конкретным конвертором.

![](https://i.imgur.com/BCnguzY.png)

![](https://i.imgur.com/00MB1h4.png)
