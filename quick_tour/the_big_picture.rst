Panorama Geral
==============

Você quer testar o Symfony2, mas só tem 10 minutos disponível? A primeira parte 
desse tutorial foi escrita para você. Aqui será explicado como começar com o Symfony2 
mostrando a estrutura de um simples projeto pronto.

Se você nunca utilizou um framework antes, você vai se sentir em casa com Symfony2.

.. index::
   pair: Sandbox; Download

Baixando e instalando o Symfony2
--------------------------------

Primeiramente, veja se você possui o PHP 5.3.2 instalado e configurado corretamente 
junto com um servidor web como o Apache.

Pronto? Vamos começar fazendo o download do Symfony2. Para comecarmos mais rápido ainda, 
vamos utilizar o "Symfony2 sandbox". Esse é um projeto onde todas as bibliotecas e 
controladores já vem incluídos; a instalação básica já está feita. A grande vantagem 
de utilizar o sandbox ao invés de outras instalações é que você pode começar a 
experimentar o Symfony2 imediatamente.

Faça o download do`sandbox`_, e descompacte no seu diretório web. Você vai ter agora 
o diretório ``sandbox/``:  

    www/ <- seu diretório web
        sandbox/ <- arquivos descompactados
            app/
                cache/
                config/
                logs/
            src/
                Application/
                    HelloBundle/
                        Controller/
                        Resources/
                vendor/
                    symfony/
            web/

.. index::
   single: Instalação; Checar

Validando a configuração
------------------------

Para evitar problemas futuramente, vamos validar se nossa configuração 
esta pronta para rodar um projeto Symfony2 corretamente acessando o endereço:

    http://localhost/sandbox/web/check.php

Leia a página com cuidado e corrija os problemas caso necessário.

Agora acesse a primeira página com Symfony2:  

    http://localhost/sandbox/web/app_dev.php/

Você deve visualizar uma página de gratificação do Symfony2.

Criando a primeira aplicação
----------------------------

O sandbox já vem com uma simples ":term:`aplicação`" Hello World e é essa aplicação 
que vamos utilizar para aprender mais sobre o Symfony2. Acesse a seguinte URL para 
ser cumprimentado pelo Symfony2 (troque Fabien pelo seu nome):

    http://localhost/sandbox/web/app_dev.php/hello/Fabien

O que está acontecendo aqui? Vamos entender a URL:

.. index:: Front Controller

* ``app_dev.php``: This is a "front controller". It is the unique entry point
  of the application and it responds to all user requests;

* ``/hello/Fabien``: This is the "virtual" path to the resource the user wants
  to access.

Sua responsabilidade como desenvolvedor é escrever um código que mapeie a requisição 
do usuário (``/hello/Fabien``) para o recurso associado (``Hello Fabien!``). 

.. index::
   single: Configuração

Configuração
~~~~~~~~~~~~

Como o sistema de rotas do Symfony2 processa seu código? Lendo alguns arquivos de configuração.

Todos os arquivos de configuração do Symfony2 podem ser escritos em PHP, XML, ou 
`YAML`_ (YAML é um simples formato que torna a descrição das configurações muito simples).

.. tip::
    
    O sandbox utiliza o YAML por padrão, mas você pode trocar para XML ou PHP 
    editando o arquivo ``app/AppKernel.php``. Você pode ir para o final dessa 
    página para ler as instruções (os tutoriais mostram as configurções 
    para todos os tipos de formatos suportados).
    
.. index::
   single: Roteamento
   pair: Configuração; Rotas

Roteamento
~~~~~~~~~~

O sistema de roteamento do Symfony2 processa a requisição lendo o arquivo de configuração:

.. configuration-block::

    .. code-block:: yaml

        # app/config/routing.yml
        homepage:
            pattern:  /
            defaults: { _controller: FrameworkBundle:Default:index }

        hello:
            resource: HelloBundle/Resources/config/routing.yml

    .. code-block:: xml

        <!-- app/config/routing.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://www.symfony-project.org/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://www.symfony-project.org/schema/routing http://www.symfony-project.org/schema/routing/routing-1.0.xsd">

            <route id="homepage" pattern="/">
                <default key="_controller">FrameworkBundle:Default:index</default>
            </route>

            <import resource="HelloBundle/Resources/config/routing.xml" />
        </routes>

    .. code-block:: php

        // app/config/routing.php
        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->addRoute('homepage', new Route('/', array(
            '_controller' => 'FrameworkBundle:Default:index',
        )));
        $collection->addCollection($loader->import("HelloBundle/Resources/config/routing.php"));

        return $collection;

As primeiras linhas do arquivo de configuração definem qual código será chamado 
quando um usuário requisitar o "``/``" resource. A parte mais interessante é a última, 
que importa outro arquivo de configuração::

.. configuration-block::

    .. code-block:: yaml

        # src/Application/HelloBundle/Resources/config/routing.yml
        hello:
            pattern:  /hello/:name
            defaults: { _controller: HelloBundle:Hello:index }

    .. code-block:: xml

        <!-- src/Application/HelloBundle/Resources/config/routing.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://www.symfony-project.org/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://www.symfony-project.org/schema/routing http://www.symfony-project.org/schema/routing/routing-1.0.xsd">

            <route id="hello" pattern="/hello/:name">
                <default key="_controller">HelloBundle:Hello:index</default>
            </route>
        </routes>

    .. code-block:: php

        // src/Application/HelloBundle/Resources/config/routing.php
        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->addRoute('hello', new Route('/hello/:name', array(
            '_controller' => 'HelloBundle:Hello:index',
        )));

        return $collection;

La vamos nós! Como você pode ver, o padrão "``/hello/:name``" (uma string começando 
com ``:name`` é um espaço reservado) é mapeado a um controlador, referenciado pelo valor ``_controller``. 

.. index::
   single: Controlador
   single: MVC; Controlador

Controladores
~~~~~~~~~~~~~

O controlador é responsável por retornar a representação do resource (na maioria dos casos um HTML) e é definido como uma classe PHP:

.. code-block:: php
   :linenos:

    // src/Application/HelloBundle/Controller/HelloController.php

    namespace Application\HelloBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    class HelloController extends Controller
    {
        public function indexAction($name)
        {
            return $this->render('HelloBundle:Hello:index.php', array('name' => $name));

            // render a Twig template instead
            // return $this->render('HelloBundle:Hello:index.twig', array('name' => $name));
        }
    }

O código é bem simples, mas vamos explicar linha por linha:

* *linha 3*: Symfony2 faz a utilização de uns novos recursos do PHP 5.3. Todos os controladores possuem os nomes propriamente reservados 
(esse espaço reservado é a primeira parte do valor da rota do ``_controller``: ``HelloBundle``).

* *linha 7*: O nome do controlador é formado a partir da 
concatenação da segunda parte do valor da rota do ``_controller``, (``Hello``), e ``Controller``. Isso extende a classe ``Controller``, 
que nos proporciona alguns atalhos (vamos ver isso ainda nesse tutorial).
  
* *linha 9*: Cada controlador é feito de algumas ações. Conforme as configurações, a página de hello é tratada pela ação ``index`` 
(a terceira parte dos valores de rota do ``controller``). Esse método recebe os valores e argumentos (``$name`` no caso) do resource.

* *linha 11*: O método ``render()`` carrega e renderiza o template (``HelloBundle:Hello:index``) com as variáveis 
sendo passadas no segundo argumento. 

O que é um :term:`bundle`? Todo código que você escreve no Symfony2 é organizado em bundles. Para o Symfony2, um bundle é um 
conjunto de arquivos (arquivos PHP, stylesheets, Javascripts, imagens, ...) que implementam um único requisito (um blog, forum, ...) 
e isso pode ser compartilhado facilmente com outros desenvolvedores. No nosso exemplo possuimos apenas um bundle, o ``HelloBundle``.

Templates
~~~~~~~~~

Os controladores renderizam o template ``HelloBundle:Hello:index.php``. Vamos entender como isso se comporta. ``HelloBundle`` é o nome do bundle, 
``Hello`` é o controlador e ``index.php`` é o arquivo do template. O template em si é feito de HTML e algumas expressões simples em PHP:

.. code-block:: html+php

    <!-- src/Application/HelloBundle/Resources/views/Hello/index.php -->
    <?php $view->extend('HelloBundle::layout.php') ?>

    Hello <?php echo $name ?>!

Parabéns! Você acaba de ver seu primeiro pedaço de código Symfony2. Não foi tão difícil, concorda? Com Symfony2 fica muito mais fácil 
e rápido de implementar websites.  

.. index::
   single: Ambiente
   single: Configuração de; Ambiente

Trabalhando com ambientes
-------------------------

Agora que você já entende um pouco como o Symfony2 funciona, vamos dar uma olhada mais detalhada no final da página; 
você vai ver uma pequena barra com os logotipos do Symfony2 e PHP. Essa barra é chamada de "Web Bebug Toolbar" e é o melhor 
amigo do desenvolvedor. É claro que essa barra não será mostrada quando você publicar sua aplicação no servidor de produção. 
É por isso que você encontrará mais controladores (``app.php``) no diretório ``web/``, prontos e otimizados para seu ambiente 
de produção. 

    http://localhost/sandbox/web/app.php/hello/Fabien

Se você configurou seu Apache com o ``mod_rewrite`` habilitado, é possível omitir o ``app.php`` da URL:

    http://localhost/sandbox/web/hello/Fabien

Nos servidores de produção você deve apontar seu diretório web para o diretório ``web/`` para garantir a segurança de sua 
instalação e obter uma URL mais bonita:

    http://localhost/hello/Fabien

Para fazer nosso servidor de produção ainda mais rápido, o Symfony2 utiliza o diretório ``app/cache/`` para manter diversos arquivos 
de cache. Quando você faz alterações no seu código ou configuração, é necessário remover os arquivos desta pasta manualmente. É 
por isso que você deve sempre utilizar o controlador de desenvolvimento (``app_dev.php``) quando não estiver no ambiente de produção. 

Últimas considerações
---------------------

Os 10 minutos acabaram. Por agora, você já pode criar suas rotas, controladores e templates. Como exercício, tente fazer alguma 
coisa mais útil do que uma aplicação do tipo "Olá mundo"! Mas, se você estiver ansioso para aprender mais sobre o Symfony2, vá para a próxima 
parte desse tutorial onde nós vamos mais afundo do sistema de templates.

.. _sandbox: http://symfony-reloaded.org/code#sandbox
.. _YAML:    http://www.yaml.org/
