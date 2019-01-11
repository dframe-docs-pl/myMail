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

 return [
     /**
      * Specify main and backup SMTP servers
      */
     'hosts' => ['primaryHostName.tld', 'backupHostName.tld'],
 
     /**
      * Enable SMTP authentication
      */
     'smtpAuth' => true,
 
     /**
      * SMTP username
      */
     'username' => 'Username@mail',
 
     /**
      * SMTP password
      */
     'password' => '',
 
     /**
      * Enable TLS encryption, `ssl` also accepted
      */
     'smtpSecure' => 'tls',
 
     /**
      * Port
      */
     'port' => 587,
 
     /**
      * Name of default sender
      */
     'senderName' => PROJECT_NAME,
 
     /**
      * Default sender's address
      */
     'senderEmail' => 'senderMail@mail'
 ];
