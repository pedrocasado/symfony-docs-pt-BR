A arquitetura
=============

Você é meu herói! Quem diria que você estaria aqui depois das três primeiras partes? 
Seus esforços serão recompensados muito brevemente. As primeiras três partes não 
mostram detalhadamente como funciona a arquitetura do framework. Como o Symfony2 se 
destaca no meio da maioria, vamos entender como tudo funciona agora.  

.. index::
   single: A estrutura de diretórios

A estrutura de diretórios
-------------------------

A estrutura de diretórios de um :term:`aplicação` do Symfony2 é bem flexível.

* ``app/``: Esse diretório contém todas as configurações da aplicação;

* ``src/``: Todo código PHP fica dentro desse diretório;

* ``web/``: Esse é o diretório web do projeto

O diretório web
~~~~~~~~~~~~~~~

O diretório web é aonde ficam todos os arquivos públicos como imagens, css e javascripts. É nesse diretório que também ficam os controladores::

    // web/app.php
    require_once __DIR__.'/../app/AppKernel.php';

    use Symfony\Component\HttpFoundation\Request;

    $kernel = new AppKernel('prod', false);
    $kernel->handle(new Request())->send();

Como qualquer controlador, o ``app.php`` utiliza a classe Kernel, ``AppKernel``, para interagir com a aplicação. 

.. index::
   single: Kernel

O diretório da aplicação
~~~~~~~~~~~~~~~~~~~~~~~~

A classe ``AppKernel`` é o principal local de configuração da aplicação, assim como todos arquivos dentro do diretório ``app/``.

Essa classe precisa obrigatoriamente implementar os quatro métodos:

* ``registerRootDir()``: Retorna o diretório root de configuração;

* ``registerBundles()``: Retorna um array com todos os bundles necessários por aquela aplicação (veja a referência ``Application\HelloBundle\HelloBundle``);

* ``registerBundleDirs()``: Retorna um array associando os namespaces com os respectivos diretórios home;  

* ``registerContainerConfiguration()``: Retorna o principal objeto de configuração (veremos detalhes adiante);

Veja a implementação padrão desses métodos para entender melhor a flexibilidade do framework.

Para fazer tudo funcionar junto, o kernel precisa de um arquivo que fica no diretório ``src/``::

    // app/AppKernel.php
    require_once __DIR__.'/../src/autoload.php';

O diretório source
~~~~~~~~~~~~~~~~~~

O arquivo ``src/autoload.php`` é responsável por carregar automaticamente todos os arquivos 
que estão na pasta ``src/``::

    // src/autoload.php
    $vendorDir = __DIR__.'/vendor';

    require_once $vendorDir.'/symfony/src/Symfony/Component/HttpFoundation/UniversalClassLoader.php';

    use Symfony\Component\HttpFoundation\UniversalClassLoader;

    $loader = new UniversalClassLoader();
    $loader->registerNamespaces(array(
        'Symfony'                        => $vendorDir.'/symfony/src',
        'Application'                    => __DIR__,
        'Bundle'                         => __DIR__,
        'Doctrine\\Common\\DataFixtures' => $vendorDir.'/doctrine-data-fixtures/lib',
        'Doctrine\\Common'               => $vendorDir.'/doctrine-common/lib',
        'Doctrine\\DBAL\\Migrations'     => $vendorDir.'/doctrine-migrations/lib',
        'Doctrine\\ODM\\MongoDB'         => $vendorDir.'/doctrine-mongodb/lib',
        'Doctrine\\DBAL'                 => $vendorDir.'/doctrine-dbal/lib',
        'Doctrine'                       => $vendorDir.'/doctrine/lib',
        'Zend'                           => $vendorDir.'/zend/library',
    ));
    $loader->registerPrefixes(array(
        'Swift_' => $vendorDir.'/swiftmailer/lib/classes',
        'Twig_'  => $vendorDir.'/twig/lib',
    ));
    $loader->register();

O ``UniversalClassLoader`` do Symfony2 é utilizado para carregar as classes que 
respeitam os `padrões`_ do PHP 5.3 ou do PEAR. Como vocês podem ver, todas as 
dependencias ficam dentro do diretório ``vendor/``, por `convenção`_. Você pode 
armazenar os arquivos aonde você quiser, globalmente no seu servidor ou localmente 
nos seus projetos.

.. index::
   single: Bundles

O sistema de bundle
-------------------

Essa seção detalha um dos maiores e mais poderosos recursos do Symfony2, o sistema de :term:`bundle`.

Um bundle é como se fosse um plugin dentro de um programa. Então por que um bundle é 
chamado de bundle e não de plugin? Porque tudo é bundle no Symfony2, desde os 
recursos core até o código que você escreve para sua aplicação. Os bundles são as 
primeiras classes geradas do Symfony2. Isso possibilita total flexibilidade para 
integrar bibliotecas/ferramentas de terceiros ou até mesmo gerar seus próprios 
bundles. Dessa maneira fica muito prático de escolher, habilitar e otimizar qualquer 
recurso que sua aplicação deve ou não utilizar.

Uma aplicação é feita de diversos bundles como definido no método ``registerBundles()`` 
da classe ``AppKernel``::

    // app/AppKernel.php
    public function registerBundles()
    {
        $bundles = array(
            new Symfony\Bundle\FrameworkBundle\FrameworkBundle(),
            new Symfony\Bundle\TwigBundle\TwigBundle(),

            // enable third-party bundles
            new Symfony\Bundle\ZendBundle\ZendBundle(),
            new Symfony\Bundle\SwiftmailerBundle\SwiftmailerBundle(),
            new Symfony\Bundle\DoctrineBundle\DoctrineBundle(),
            //new Symfony\Bundle\DoctrineMigrationsBundle\DoctrineMigrationsBundle(),
            //new Symfony\Bundle\DoctrineMongoDBBundle\DoctrineMongoDBBundle(),

            // register your bundles
            new Application\HelloBundle\HelloBundle(),
        );

        if ($this->isDebug()) {
            $bundles[] = new Symfony\Bundle\WebProfilerBundle\WebProfilerBundle();
        }

        return $bundles;
    }

De acordo com o ``HelloBundle`` que vimos anteriormente, perceba que o kernel 
também habilita o ``FrameworkBundle``, ``DoctrineBundle``, ``SwiftmailerBundle``, e ``ZendBundle``.
Todos esses bundles fazem parte do core do framework.

Cada bundle pode ser personalizado através dos arquivos de configuração escritos em 
YAML, XML ou PHP. Veja um arquivo de configuração:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        app.config:
            charset:       UTF-8
            error_handler: null
            csrf_secret:   xxxxxxxxxx
            router:        { resource: "%kernel.root_dir%/config/routing.yml" }
            validation:    { enabled: true, annotations: true }
            templating:
                #assets_version: SomeVersionScheme
            session:
                default_locale: en
                lifetime: 3600

        ## Twig Configuration
        #twig.config:
        #    auto_reload: true

        ## Doctrine Configuration
        #doctrine.dbal:
        #    dbname:   xxxxxxxx
        #    user:     xxxxxxxx
        #    password: ~
        #doctrine.orm: ~

        ## Swiftmailer Configuration
        #swiftmailer.config:
        #    transport:  smtp
        #    encryption: ssl
        #    auth_mode:  login
        #    host:       smtp.gmail.com
        #    username:   xxxxxxxx
        #    password:   xxxxxxxx

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <app:config csrf-secret="xxxxxxxxxx" charset="UTF-8" error-handler="null">
            <app:router resource="%kernel.root_dir%/config/routing.xml" />
            <app:validation enabled="true" annotations="true" />
            <app:session default-locale="en" lifetime="3600" />
        </app:config>

        <!-- Twig Configuration -->
        <!--
        <twig:config auto_reload="true" />
        -->

        <!-- Doctrine Configuration -->
        <!--
        <doctrine:dbal dbname="xxxxxxxx" user="xxxxxxxx" password="" />
        <doctrine:orm />
        -->

        <!-- Swiftmailer Configuration -->
        <!--
        <swiftmailer:config
            transport="smtp"
            encryption="ssl"
            auth_mode="login"
            host="smtp.gmail.com"
            username="xxxxxxxx"
            password="xxxxxxxx" />
        -->

    .. code-block:: php

        // app/config/config.php
        $container->loadFromExtension('app', 'config', array(
            'charset'       => 'UTF-8',
            'error_handler' => null,
            'csrf-secret'   => 'xxxxxxxxxx',
            'router'        => array('resource' => '%kernel.root_dir%/config/routing.php'),
            'validation'    => array('enabled' => true, 'annotations' => true),
            'templating'    => array(
                #'assets_version' => "SomeVersionScheme",
            ),
            'session' => array(
                'default_locale' => "en",
                'lifetime' => "3600",
            ),
        ));

        // Twig Configuration
        /*
        $container->loadFromExtension('twig', 'config', array('auto_reload' => true));
        */

        // Doctrine Configuration
        /*
        $container->loadFromExtension('doctrine', 'dbal', array(
            'dbname'   => 'xxxxxxxx',
            'user'     => 'xxxxxxxx',
            'password' => '',
        ));
        $container->loadFromExtension('doctrine', 'orm');
        */

        // Swiftmailer Configuration
        /*
        $container->loadFromExtension('swiftmailer', 'config', array(
            'transport'  => "smtp",
            'encryption' => "ssl",
            'auth_mode'  => "login",
            'host'       => "smtp.gmail.com",
            'username'   => "xxxxxxxx",
            'password'   => "xxxxxxxx",
        ));
        */

Cada entrada no ``app.config`` define a configuração para um bundle.

Cada :term:`ambiente` pode sobreescrever as configurações padrões
utilizando um arquivo de configuração específico:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config_dev.yml
        imports:
            - { resource: config.yml }

        app.config:
            router:   { resource: "%kernel.root_dir%/config/routing_dev.yml" }
            profiler: { only_exceptions: false }

        webprofiler.config:
            toolbar: true
            intercept_redirects: true

        zend.config:
            logger:
                priority: debug
                path:     %kernel.logs_dir%/%kernel.environment%.log

    .. code-block:: xml

        <!-- app/config/config_dev.xml -->
        <imports>
            <import resource="config.xml" />
        </imports>

        <app:config>
            <app:router resource="%kernel.root_dir%/config/routing_dev.xml" />
            <app:profiler only-exceptions="false" />
        </app:config>

        <webprofiler:config
            toolbar="true"
            intercept-redirects="true"
        />

        <zend:config>
            <zend:logger priority="info" path="%kernel.logs_dir%/%kernel.environment%.log" />
        </zend:config>

    .. code-block:: php

        // app/config/config_dev.php
        $loader->import('config.php');

        $container->loadFromExtension('app', 'config', array(
            'router'   => array('resource' => '%kernel.root_dir%/config/routing_dev.php'),
            'profiler' => array('only-exceptions' => false),
        ));

        $container->loadFromExtension('webprofiler', 'config', array(
            'toolbar' => true,
            'intercept-redirects' => true,
        ));

        $container->loadFromExtension('zend', 'config', array(
            'logger' => array(
                'priority' => 'info',
                'path'     => '%kernel.logs_dir%/%kernel.environment%.log',
            ),
        ));

Como vimos no trecho acima, a aplicação é feita de bundles como definidos no método 
``registerBundles()``. Como o Symfony2 sabe aonde procurar pelos bundles? O Symfony2 
é flexível o suficiente para dar conta disso. O método ``registerBundleDirs()`` 
retorna obrigatoriamente um array que mapeia todos os namespaces para qualquer 
diretório válido (local ou global)::

    public function registerBundleDirs()
    {
        return array(
            'Application'     => __DIR__.'/../src/Application',
            'Bundle'          => __DIR__.'/../src/Bundle',
            'Symfony\\Bundle' => __DIR__.'/../src/vendor/symfony/src/Symfony/Bundle',
        );
    }

Então, quando você utilizar o ``HelloBundle`` em um controlador ou template, o Symfony2 
vai procurar por ele nos diretórios referenciados.

Consegue perceber agora o quanto o Symfony2 é flexível? Compartilhe seus bundles 
entre suas aplicações ou armazene eles localmente ou globalmente. Fica a seu critério.

.. index::
   single: Bibliotecas de terceiros

Utilizando bibliotecas de terceiros
-----------------------------------

Há casos em que sua aplicação pode precisar de bibliotecas de terceiros. Esses arquivos 
devem ficam no diretório ``src/vendor/``. Esse diretório já possui algumas bibliotecas 
utilizadas pelo Symfony2 como o SwiftMailer, Doctrine ORM, Twig e algumas classes 
do Zend Framework.

.. index::
   single: Configuração do cache
   single: Logs

Cache e Logs
------------

Symfony2 é provavelmente o framework mais rápido que há no momento. Como ele pode 
ser tão rápido já que é necessário interpretar dezenas de arquivos YAML e XML para 
cada requisição? Isso faz parte do sistema de cache. A configuração da aplicação 
é interpretada somente na primeira requisição e é então compilada para um arquivo 
PHP que fica armazenado no diretório ``cache/`` da aplicação. No ambiente de desenvolvimento, 
o Symfony2 é esperto o suficiente para renovar o cache quando você alterar algum arquivo.
No ambiente de produção, quando algum código ou configuração for alterado, a limpeza do cache é de sua responsabilidade.

Quando desenvolvems uma aplicação web, algumas coisas podem dar errado. Os arquivos 
de log ficam no diretório ``logs/`` da aplicação. Eles armazenam todas as requisições 
e ajudam na correção de problemas.

.. index::
   single: CLI
   single: Linha de comando

A interface de linha de comando
-------------------------------

Cada aplicação vem com uma interface de linha de comando (``console``) que ajuda 
a controlar sua aplicação. Essa interface possui diversos comandos para automatizar 
sua produtividade e evitar trabalhos repetitivos.
 
Execute sem nenhum argumento para entender melhor:

.. code-block:: bash

    $ php app/console

A opção ``--help`` te ajuda a descobrir os comandos existentes:

.. code-block:: bash

    $ php app/console router:debug --help

Últimas considerações
---------------------

Posso estar enganado, mas depois de ler essa parte você já deve ter percebido que 
o Symfony2 serve para você. Tudo no Symfony2 foi feito para funcionar do seu jeito. 
Fique a vontade para renomear ou remover diretórios de acordo com seu gosto.

Esse é o guia rápido. Para aprender mais e se tornar um expert em Symfony2, não deixe 
de ler os `guias`_ oficiais e escolha o tópico que quiser. 

.. _padrões:  http://groups.google.com/group/php-standards/web/psr-0-final-proposal
.. _convenção: http://pear.php.net/
.. _guias:     http://www.symfony-reloaded.org/learn