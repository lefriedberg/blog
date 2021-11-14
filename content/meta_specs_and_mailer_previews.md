+++
title = "Meta Specs and Mailer Previews"
date = 2021-08-21
draft = true
[extra]
show_date = true
+++

Summary

## On mailer previews
`https://guides.rubyonrails.org/action_mailer_basics.html#previewing-emails`

Mailer previews are an excellent tool that rails provides to demonstrate how emails look in our application. They are useful for 
1. Developing mailers
1. Editing existing mailers
1. Understanding the mailers a given system sends

The third use case is relevant to lots of non-engineering stakeholders. It's very valuable if product, design and business stakeholders can see sample mailers.

By default mailer previews are only hosted in the development environment. This makes them not particularly useful to non-engineering stakeholders.

I like to host mailers in the admin app so I set `config.action_mailer.show_previews = true` and then put them behind authentication.

However, by hosting our mailer previews, I uncovered a couple problems:
1. Some of them raised errors because the code they depended on had changed.
1. Some of them used database access or FactoryBot for their sample data.

## On meta specs

Meta specs are useful for validating invariants of our application.

While I like to use RSpec as my testing library

In conculusion, we have.

```ruby
RSpec.describe "Mailer Previews", type: :request do
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
