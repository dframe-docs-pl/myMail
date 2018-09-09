.. title:: MyMail -  Wrapper dla wysyłki maili

.. meta::
    :description: MyMail -  Wrapper dla wysyłki maili - dframeframework.com
    :keywords: php, mailing, php, php7, send mail, mails, maile, smtp, imap, mail wrapper, dframe
    
Biblioteka myMail to prosty wrapper tym razem dla phpmailer. W prosty sposób ustawiasz dane do połaczenia w configu.

Instalacja
----------

Z poziomu konsoli bash wykonaj polecenie composera*

.. code-block:: bash

 $ composer require dframe/mymail

Albo pobierz ręcznie https://github.com/dframe/myMail/releases

Konfiguracja
----------

.. code-block:: php

 return array(
    'Hosts' => array('primaryHostName.tld', 'backupHostName.tld'),    // Specify main and backup SMTP servers
    'SMTPAuth' => true,    // Enable SMTP authentication
    'Username' => 'Username@mail',    // SMTP username
    'Password' => '',    // SMTP password
    'SMTPSecure' => 'tls',    // Enable TLS encryption, `ssl` also accepted
    'Port' => 587,    // Port

    'setMailTemplateDir' => './View/templates/mail',
    'smartyHtmlExtension' => '.html.php',    // Default '.html.php'
    'smartyTxtExtension' => '.txt.php',    //Default '.txt.php'
    'fileExtension' => '.html.php',    

    'senderName' => PROJECT_NAME,    //Name of default sender
    'senderMail' => 'senderMail@mail'    //Default sender's address
 );
