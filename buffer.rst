.. title:: MyMail - Kolejkowanie maili

.. meta::
    :description: MyMail - Kolejkowanie maili - dframeframework.com
    :keywords: php, mailing, php, e-mail, php7, send mail, buffer, kolejkowanie, smtp, imap, mail wrapper, dframe

Stworzenie buforu pozwala nam stworzyć kolejkowanie maili dzięki czemu możemy wraz z dframe/database ustawić by był limit wysyłanych maili na godzinę. W tym celu gdy posiadamy już pobranego i zainstalowanego frameworka, tworzymy model o nazwie mail.php

**Model/Mail.php**

.. code-block:: php

 <?php
 
 namespace Model;
 
 use Dframe\Config;
 use Dframe\MyMail\MyMail;
 
 class MailModel extends Model
 {
 
     /**
      * @var array
      */
     protected $buffer;
 
     /**
      * Init function
      */
     public function init()
     {
         $this->buffer = [];
     }
 
 
     /**
      * @param array $whereObject
      *
      * @return init
      */
     public function mailsCount($whereObject = [])
     {
 
         $query = $this->db->prepareQuery('SELECT COUNT(*) AS `count` FROM `mails`');
         $query->prepareWhere($whereObject);
 
         $row = $this->db->pdoQuery($query->getQuery(), $query->getParams())->result();
         return $row['count'];
     }
 
 
     /**
      * @param int    $start
      * @param int    $limit
      * @param array  $whereObject
      * @param string $order
      * @param string $sort
      *
      * @return array
      */
     public function mails($start, $limit, $whereObject, $order = 'id', $sort = 'DESC')
     {
 
         $query = $this->db->prepareQuery(
             'SELECT mail.*, 
                  `users`.`id`
              FROM  mail 
              LEFT JOIN users ON `mail`.`mail_address` = `users`.`email`'
         );
         $query->prepareWhere($whereObject);
         $query->prepareOrder($order, $sort);
         $query->prepareLimit($limit, $start);
 
         $results = $this->db->pdoQuery($query->getQuery(), $query->getParams())->results();
         return $this->methodResult(true, ['data' => $results]);
     }
 
     /**
      * @param array  $address
      * @param string $subject
      * @param string $body
      * @param string $sender
      * @param array  $attachmentsIds
      *
      * @return array
      */
     public function addToBuffer(array $address, $subject, $body, $sender = '', array $attachmentsIds = [])
     {
 
         $DateUTC = new \DateTime("now", new \DateTimeZone("UTC"));
 
         $mailEntry = [
             'mail_name' => $address['name'],
             'mail_address' => $address['mail'],
             'mail_subject' => $subject,
             'mail_enqueued' => time(),
             'mail_body' => $body,
             'mail_sender' => $sender,
             'mail_status' => 0,
             'mail_buffer_date' => $DateUTC->format('Y-m-d H:i:s')
         ];
 
         //if($attachment != false){
         //    $mailEntry['mail_attachments_ids'] = $attachmentsIds;
         //}
 
         $this->buffer[] = $mailEntry;
 
         return $this->methodResult(true);
     }
 
     /**
      * @return array
      */
     public function execute()
     {
         //Pusty
         if (count($this->buffer) == 0) {
             return $this->methodResult(false, ['response' => 'Buffer is empty']);
         }
 
         $i = 0;
         try {
 
             foreach ($this->buffer as $key => $value) {
 
                 $buffer = [
                     'mail_name' => $value['mail_name'],
                     'mail_address' => $value['mail_address'],
                     'mail_subject' => $value['mail_subject'],
                     'mail_enqueued' => $value['mail_enqueued'],
                     'mail_body' => $value['mail_body'],
                     'mail_sender' => $value['mail_sender'],
                     'mail_status' => $value['mail_status'],
                     'mail_buffer_date' => $value['mail_buffer_date']
                 ];
 
                 $insertResult = $this->db->insert('mails', $buffer, true)->getLastInsertId();
                 if ($insertResult > 0) {
                     throw new Exception("Filed to add mail", 1);
                 }
 
                 // Adding attachments do mysql
                 // if (isset($buffer['mail_attachments_ids'])){
 
                 //     $attachments = array();
                 //     foreach ($buffer['mail_attachments_ids'] as $key2 => $value2) {
                 //         $attachments[] = array(
                 //             'mail_id' => $insertResult,
                 //             'file_id' => $value['mail_attachments_ids']
                 //         );
                 //     }
 
                 //     $insertAttachmentsResult = $this->db->insertBatch('mails_attachments', $attachments)->getLastInsertId();
                 //     if(count($insertAttachmentsResult)){
                 //         throw new Exception("Filed to add attachment", 1);
                 //     }
                 // }
 
                 $i++;
             }
 
         } catch (Exception $e) {
             return $this->methodResult(false, ['response' => $e->getMessage()]);
         }
 
         if (!count($i)) {
             return $this->methodResult(false, ['response' => 'Unable to add mails to spooler']);
         }
 
         $this->buffer = [];
         return $this->methodResult(true);
     }
 
     /**
      * @param int $amount
      *
      * @return mixed
      */
     public function sendMails($amount = 20)
     {
 
         $amount = (int)$amount;
         if ($amount <= 0) {
             return $this->methodResult(false, 'Incorrect amount');
         }
 
         $emailsToSend = $this->db->pdoQuery(
             'SELECT * 
              FROM `mails` WHERE `mail_status` = ?
              ORDER BY `mail_enqueued` ASC
              LIMIT ?', ['0', $amount])->results();
 
         $data = ['sent' => 0, 'failed' => 0, 'errors' => []];
         $return = true;
 
         $MyMail = new MyMail();
         $MyMail->mail->isSMTP();
         $MyMail->mail->SMTPOptions = [
             'ssl' => [
                 'verify_peer' => false,
                 'verify_peer_name' => false,
                 'allow_self_signed' => true
             ]
         ];
         //$MyMail->mail->SMTPDebug  = 2; // enables SMTP debug information (for testing)
         // 1 = errors and messages
         // 2 = messages only
         $MyMail->mail->SMTPSecure = false;
 
         foreach ($emailsToSend as $email) {
 
             $DateUTC = new \DateTime("now", new \DateTimeZone("UTC"));
             try {
 
                 //$mailsAttachments = $this->db->pdoQuery('SELECT * FROM `mails_attachments` LEFT JOIN files ON mails_attachments.file_id = files.file_id WHERE mail_id = ?', array($email['mail_id']))->results();
                 //if (count($mailsAttachments) > 0) {
 
                 //foreach ($mailsAttachments as $key => $attachment) {
                 //
                 //    $FileStorage = new \Dframe\FileStorage\Storage($this->loadModel('FileStorage/Drivers/DatabaseDriver'));
                 //
                 //    $sourceAdapter = $attachment['file_adapter'].'://'. $attachment['file_path'];
                 //
                 //    if ($FileStorage->manager->has($sourceAdapter)) {
                 //        // Retrieve a read-stream
                 //        $stream = $FileStorage->manager->readStream($sourceAdapter);
                 //
                 //        $contents = stream_get_contents($stream);
                 //        $MyMail->mail->addStringAttachment($contents, end(explode('/', $attachment['file_path'])));
                 //        fclose($stream);
                 //
                 //    } else {
                 //        throw new \Exception("Brak załacznika", 1);
                 //    }
                 //
                 //
                 //}
 
                 //}
 
                 $addAddress = ['mail' => $email['mail_address'], 'name' => $email['mail_name']];
                 $sendResult = $MyMail->send($addAddress, $email['mail_subject'], $email['mail_body']);
 
                 $this->db->update('mails', ['mail_sent' => time(), 'mail_status' => '1', 'mail_send_date' => $dateUTC->format('Y-m-d H:i:s')], ['mail_id' => $email['mail_id']]);
                 $data['sent']++;
 
 
             } catch (\Exception $e) {
                 $data['errors'][] = $e->getMessage();
             }
 
             if (!isset($sendResult)) {
                 $data['failed']++;
                 $return = false;
                 continue;
             }
 
         }
 
         //var_dump($data);
         return $this->methodResult($return, $data);
     }
 
     /**
      * @return array
      */
     public function clear()
     {
         $this->db->truncate('mails');
         return $this->methodResult(true);
     }
 
 }


Cron jest odpalany z poziomu konsoli, służy do cyklicznego sprawdzania czy jest coś do wysłania. Jeśli natrafi na wpis w bazie z mailem to postara się go nam wysłać.

**bin/SmallCron.php**

.. code-block:: php

 <?php
 use Dframe\Router\Response;

 set_time_limit(0);
 ini_set('max_execution_time', 0);
 date_default_timezone_set('Europe/Warsaw');

 require_once __DIR__ . '/../../vendor/autoload.php';
 require_once __DIR__ . '/../../web/config.php';

 /**
  * Self Aonymous Cron class
  */
 return (new class() extends \Dframe\Cron\Task
 {

     /**
      * @return Response
      */
     public function init()
     {
         $cron = $this->inLock('mail', [$this->loadModel('Mail'), 'sendMails'], []);
         if ($cron['return'] == true) {
             $mail = $cron['response'];
             return Response::renderJSON(['code' => 200, 'message' => 'Cron Complete', 'data' => ['mail' => ['data' => $mail['response']]]]);
         }

         return Response::renderJSON(['code' => 403, 'message' => 'Cron in Lock'])->status(403);

     }
 }
 )->init()->display();


Ostatnim naszym elementem jest kod dodający do bazy. Metoda |addToBuffer| służy do zbierania listy mailingowej a następnie po skończeniu wykonujemy |execute| która dodaje do bazy listę i ją kolejkuje.

.. code-block:: php

 $MailModel = $this->loadModel('Mail');
 $MailModel->addToBuffer(['name' => 'NameRespondent', 'mail' => 'respondent@Email_respondent'], 'Subjectname', $body);
 $execute = $mailModel->execute();

.. |addToBuffer| cCode:: $MailModel->addToBuffer
.. |execute| cCode:: $MailModel->execute
