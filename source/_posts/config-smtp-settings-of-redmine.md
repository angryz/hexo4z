title: 配置Redmine通过smtp服务发送邮件通知
date: 2013-08-16 15:43:17
tags: tools
categories: System
---

In order to deliver emails to users of Redmine, we need to complete the email configuration.

Email configuration of Redmine is contained in the `configuration.yml` file.

```bash
vim /var/www/redmine/config/configuration.yml
```

Let's see a example email configuration first.

```yml
production:
  email_delivery:
    delivery_method: :smtp
    smtp_settings:
      address: smtp.mailserver.com
      port: 25
      domain: mailserver.com
      authentication: :login
      user_name: 'redmine@mydomain.com'
      password: 'mypassword'
```

Because Redmine is developed with Ruby on Rails, so it's email configuration is based on [Action Mailer Configuration](http://guides.rubyonrails.org/action_mailer_basics.html#action-mailer-configuration) of Ruby on Rails.

*delivery_method*
The mail transport method to be used.
Valid settings:
* :smtp
* :sendmail
* :async_smtp
* :async_sendmail

The `:async_smtp` and `:async_sendmail` use asynchronous sends, which means Redmine does not wait for the email to be sent to display the next page. See [Asynchronous Email Delivery](http://redmineblog.com/articles/asynchronous-email-delivery/) for more details. Some SMTP servers have delay period before processing takes place as anti-spam feature, during which time synchronous method will block Redmine ([10 seconds](http://answers.bitnami.org/questions/4421/updating-an-issue-in-redmine-takes-10-seconds) could be default value).

*address*
The SMTP server address to be used.

*port*
The SMTP server port to be used.

*domain*
You can specify your HELO domain in this setting.

*authentication*
The type of authentication method expected by your service provider.
Valid settings:
* nil (or omit the key) for no authentication
* :plain
* :login
* :cram_md5
(note: if you set this to `nil` or omit it, you must not include the `user_name` and `password` settings)

*user_name*
If your mail server requires authentication, set the username in this setting.

*password*
If your mail server requires authentication, set the password in this setting.

And if you are using a TLS-requiring mail server, you'll have to add this setting:
*enable_starttls_auto*
Valid Settings:
* true
* false
Set this to `true` if you are using a TLS-requiring mail server.

Take GMail for example:

```yml
production:
  email_delivery:
    delivery_method: :smtp
    smtp_settings:
      enable_starttls_auto: true
      address: smtp.gmail.com
      port: 587
      domain: smtp.gmail.com
      authentication: :plain
      user_name: 'my_google_id@gmail.com'
      password: 'mypassword'
```

## More examples

### No Authentication

```yml
production:
  email_delivery:
    delivery_method: :smtp
    smtp_settings:
      address: smtp.mailserver.com
      port: 25
      domain: mydomain.com
```

### Using __sendmail__ command
For unix system command: `/usr/sbin/sendmail`
```yml
production:
  email_delivery:
    delivery_method: :sendmail
```

