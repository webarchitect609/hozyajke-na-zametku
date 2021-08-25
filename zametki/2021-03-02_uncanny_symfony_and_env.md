Странности Symfony в работе с переменными окружения
===================================================

###### 2021-03-02

Странность заключается в том, что в мире Linux принято пользоваться переменными окружения, но их отсутствие не приводит
к фатальным последствиям. Грубо говоря, я могу вызвать

```bash
composer install
```

, а могу задать переменную окружения, которая повлияет на работу программы

```bash
COMPOSER_MEMORY_LIMIT=-1 composer install
```

Symfony же почему-то считает, что если в `services.yml` написано

```yaml
$sslCertFilename: '%env(APPLE_PAY_SSL_CERT)%'
```

, то за отсутствие `APPLE_PAY_SSL_CERT` надо карать фатальной ошибкой. Тем не менее есть способ это легко вылечить.
Способ я только что нашёл и опробовал, поэтому может оказаться, что это не идеальное решение.

Итак, существует `\Symfony\Component\DependencyInjection\EnvVarProcessorInterface` и его реализация по умолчанию
`\Symfony\Component\DependencyInjection\EnvVarProcessor`. Если её завернуть в свою и научить заменять несуществующие
переменные окружения значениями по умолчанию

```php
use Closure;
use Symfony\Component\DependencyInjection\EnvVarProcessor;
use Symfony\Component\DependencyInjection\EnvVarProcessorInterface;

/**
 * Class OptionalEnvVarProcessor
 *
 * Обслуживает необязательные переменные окружения, которые заменяются на значения по умолчанию, если не заданы.
 *
 * @package Adv\AppBundle\EnvVarProcessor
 */
class OptionalEnvVarProcessor extends EnvVarProcessor implements EnvVarProcessorInterface
{
    protected function getNonStrictVars(): array
    {
        return [
            'APPLE_PAY_SSL_CERT'    => 'web/local/php_interface/apple-pay/merchant_id.cer.pem',
            'APPLE_PAY_SSL_KEY'     => 'web/local/php_interface/apple-pay/merchant_id.key.pem',
            'APPLE_PAY_KEY_PAS'     => '',
            'MCDONALDS_CMS_API_URL' => 'https://mcdonalds-cms-api.adv.ru',
        ];
    }

    /**
     * @inheritDoc
     */
    public function getEnv($prefix, $name, Closure $getEnv)
    {
        /*
         * Если переменная не задана и является необязательной,
         * то вернуть значение по умолчанию.
         */
        if (false === getenv($name) && array_key_exists($name, $this->getNonStrictVars())) {
            return $this->getNonStrictVars()[$name];
        }

        return parent::getEnv($prefix, $name, $getEnv);
    }
}
```

, то философия Linux начинает соблюдаться, а введение новой переменной окружения на проекте не вынуждает срочно бежать
по всем зонам и заводить везде необходимую переменную.
