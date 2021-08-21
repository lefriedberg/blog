+++
title = "Testing in Production"
date = 2020-06-19
draft = false
[extra]
show_date = true
+++

Test of new site.

```ruby
class Testing::File
  class << self
    def temp(contents, extension: nil)
      Tempfile.new(["temp_file", extension]).tap do |temp|
        temp.write(contents)
        temp.rewind
      end
    end
  end
end
```
