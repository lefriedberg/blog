+++
title = "Mailer Previews and Meta Specs"
date = 2021-08-21
draft = true
[extra]
show_date = true
+++

Summary

## On mailer previews

Mailer previews are a tool in rails to ...

## On meta specs

While I like to use RSpec as my testing library

In conculusion, we have.

```ruby
RSpec.describe "Mailer Preview", type: :request do
  before do
    stub_const(
      "FactoryBot",
      "FactoryBot should not be used in previews as it cannot be used in production"
    )
  end

  ActionMailer::Preview.all.each do |preview_class|
    preview_class.emails.each do |email|
      it "#{preview_class}##{email} can loads without error" do
        get "/rails/mailers/#{preview_class.preview_name}/#{email}"
        expect(response).to have_http_status(200)
      end
    end
  end
end
```
