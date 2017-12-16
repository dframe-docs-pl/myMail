Bibliotekę można stosować niezależnie tzn nie jest konieczne posiadanie framerowka. Dostęp do klasy phpMailer mamy za pośrednictwem zmiennej |mail| Poniższy przykład wysłania maila odrazu do adresata.

.. code-block:: php

 use Dframe\MyMail\MyMail;
 
 require_once __DIR__ . '/../vendor/autoload.php';
 $config = require_once 'config/config.php'; 
 
 $mail = new MyMail(); // Załadowanie Configu
 $mail->mailObject->isSMTP();
 $mail->mailObject->SMTPOptions = array(
     'ssl' => array(
         'verify_peer' => false,
         'verify_peer_name' => false,
         'allow_self_signed' => true
     )
 );
 //$mail->SMTPDebug  = 2; // enables SMTP debug information (for testing)
                        // 1 = errors and messages
                        // 2 = messages only
 $mail->mailObject->SMTPSecure = false;
 
 $addAddress = array('mail' => 'adres@email', 'name' => 'titleFrom'); // Adresy na jakie ma wysłać
 
 try {
     $mail->send($addAddress, 'Test Mail', $body);
 
 } catch (Exception $e) {
     echo $e->getMessage();
     
 }

.. |mail| cCode:: $mail->mailObject 
