.. title:: MyMail - Wysyłka maili

.. meta::
    :description: MyMail - Wysyłka maili
    :keywords: php, mailing, php, php7, send mail, buffer, queuing, smtp, imap, mail wrapper, dframe

Bibliotekę można stosować niezależnie tzn nie jest konieczne posiadanie framerowka. Dostęp do klasy phpMailer mamy za pośrednictwem zmiennej |mail| Poniższy przykład wysłania maila odrazu do adresata.

.. code-block:: php

 use Dframe\MyMail\MyMail;
 
 require_once __DIR__ . '/../vendor/autoload.php';
 $config = require_once 'config/config.php';
 
 $MyMail = new MyMail(); // Załadowanie Configu
 $MyMail->mail->isSMTP();
 $MyMail->mail->SMTPOptions = [
     'ssl' => [
         'verify_peer' => false,
         'verify_peer_name' => false,
         'allow_self_signed' => true
     ]
 ];
 
 /**
   * Enables SMTP debug information (for testing)
   * 1 = errors and messages
   * 2 = messages only
   */         
 //$mail->SMTPDebug  = 2;
 $MyMail->mail->SMTPSecure = false;
 
 $addAddress = ['mail' => 'adres@email', 'name' => 'titleFrom']; // Adresy na jakie ma wysłać
 
 try {
     $MyMail->send($addAddress, 'Test Mail', $body);
 
 } catch (Exception $e) {
     echo $e->getMessage();
 
 }

.. |mail| cCode:: $MyMail->mail 
