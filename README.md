# Symfony Dynamic Mailer

This repository provides an implementation of a dynamic mailer in Symfony. It allows you to send emails using the Symfony Mailer component with dynamic configurations.

Installation

To install the required packages, run the following commands:

```composer require symfony/mailer```

```composer require symfony/messenger```

# Directory Structure

The following files are part of this implementation:

src/Controller/UserSendEmailAction.php
src/Messages/NewUserByEmail.php
src/Messages/DynamicMailer.php
src/Messages/NewUserByEmailHandler.php

Usage

# Controller

The UserSendEmailAction controller dispatches a NewUserByEmail message to the message bus.

```
<?php

declare(strict_types=1);

namespace App\Controller;

use App\Controller\Base\AbstractController;
use App\Messages\NewUserByEmail;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Messenger\MessageBusInterface;

class UserSendEmailAction extends AbstractController
{
    public function __invoke(MessageBusInterface $messageBus): Response
    {
        $message = new NewUserByEmail('example@gmail.com', 'message');

        $messageBus->dispatch($message);

        return $this->responseEmpty();
    }
}
```

# Message Class

The NewUserByEmail class represents the email message that will be sent. It includes email, message content, subject, and SMTP configuration details.

```
<?php
declare(strict_types=1);

namespace App\Messages;

readonly class NewUserByEmail
{
    public function __construct(
        private string $email,
        private string $message,
        private string $subject = 'subject',
        private string $form = 'example@gmail.com',
        private string $smtpPort = '587',
        private string $smtpHost = 'example@gmail.com:password@smtp.gmail.com',
    )
    {
    }

    public function getSmtpHost(): string
    {
        return $this->smtpHost;
    }

    public function getSmtpPort(): string
    {
        return $this->smtpPort;
    }

    public function getEmail(): string
    {
        return $this->email;
    }

    public function getMessage(): string
    {
        return $this->message;
    }

    public function getForm(): string
    {
        return $this->form;
    }

    public function getSubject(): string
    {
        return $this->subject;
    }

}
```

# Dynamic Mailer

The DynamicMailer class implements MailerInterface and sends emails using a dynamically configured transport.

```
<?php
declare(strict_types=1);

namespace App\Messages;

use Symfony\Component\Mailer\Envelope;
use Symfony\Component\Mailer\Mailer;
use Symfony\Component\Mailer\MailerInterface;
use Symfony\Component\Mailer\Transport\TransportInterface;
use Symfony\Component\Mime\RawMessage;

class DynamicMailer implements MailerInterface
{
    private Mailer $mailer;

    public function __construct(TransportInterface $transport)
    {
        $this->mailer = new Mailer($transport);
    }

    public function send(RawMessage $message, Envelope $envelope = null): void
    {
        $this->mailer->send($message, $envelope);
    }
}
```

# Message Handler
The NewUserByEmailHandler handles the NewUserByEmail message and sends the email using the DynamicMailer.

```
<?php
declare(strict_types=1);

namespace App\Messages;

use Symfony\Component\Mailer\Exception\TransportExceptionInterface;
use Symfony\Component\Mailer\Transport;
use Symfony\Component\Messenger\Attribute\AsMessageHandler;
use Symfony\Component\Mime\Email;

#[AsMessageHandler]
readonly class NewUserByEmailHandler
{
    /**
     * @throws TransportExceptionInterface
     */
    public function __invoke(NewUserByEmail $message): void
    {
        $email = (new Email())
            ->from($message->getForm())
            ->to($message->getEmail())
            ->subject($message->getSubject())
            ->html($message->getMessage());

        $this->connectedToDSN($message)->send($email);
    }

    private function connectedToDSN(NewUserByEmail $message): DynamicMailer
    {
        $dsn = 'smtp://' . $message->getSmtpHost() . ':' . $message->getSmtpPort();

        return new DynamicMailer(Transport::fromDsn($dsn));
    }
}
```
