# Fake E-mail Address

A scammer can use an identical E-mail address to those of famous companies or institutions to send e-mails to victims. Eg., impersonating OpenSea to send phishing E-mails.



SMTP(Simple Mail Transfer Protocol) is the protocol that defines how E-mail works. It's an application-level protocol based on TCP with three phases: Establishment of connection, data transmission and connection termination.

There are some commands in SMTP, eg.:

| Command   | Description                                           |
| --------- | ----------------------------------------------------- |
| HELO      | Identify the domain name of the sending host to SMTP. |
| MAIL FROM | Specify the sender of the mail.                       |
| RCPT TO   | Specify the recipients of the mail.                   |
| DATA      | Define the following information as data.             |
| QUIT      | End an SMTP connection.                               |

A scammer can assign `HELO` to any E-mail address he wants to impersonate.

By checking the sender's IP we can determine it's fake in some extent. But this method can be very difficult for ordinary users, who are more vulnerable to the fake E-mail scam.

