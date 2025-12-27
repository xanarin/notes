# Apprise flexible notification system

[Apprise](https://github.com/caronc/apprise) is a cool project that allows you to send notifications through a lot of different mediums. 


## Email Hosted by Migadu

To have `apprise` send email notifications from `as@domain.com` to `user@domain.com` where `domain.com` is hosted by Migadu and the password for the `AS@domain.com` account is `X123456789ABCD`, the command is:
```sh
$ apprise -b 'This is the body' -t 'This is the Title' 'mailtos://X123456789ABCD@smtp.migadu.com/user@domain.com?from=Apprise Test<as@domain.com>&user=as@domain.com'
```
