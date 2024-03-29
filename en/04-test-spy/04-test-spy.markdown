# Test Spy #

## Objective ##

 - Using Stubs with Test Spy in Ruby

### Problem ###

 I came across a problem during testing. I had to test the cookie setting logic of my controllers. It was straightforward to test that the cookie was set for the happy path. For the alternative scenario it became tricky to test because RSpec and Rails framework did not play that well together. I even read Devise Rails plugin code to see how Jose Valim handled cookie related problems during testing. No luck. One solution I found was on Stackoverflow: How do I test cookie expiry?

app/controllers/widget_controller.rb

```ruby
...
def index
    cookies[:expiring_cookie] = { :value   => 'All that we see or seem...', 
                                  :expires => 1.hour.from_now }
end
...
```

spec/controllers/widget_controller_spec.rb

```ruby
...
it "sets the cookie" do
  get :index
  response.cookies['expiring_cookie'].should eq('All that we see or seem...')
end

it "sets the cookie expiration" do
  stub_cookie_jar = HashWithIndifferentAccess.new
  controller.stub(:cookies) { stub_cookie_jar }

  get :index
  expiring_cookie = stub_cookie_jar['expiring_cookie']
  expiring_cookie[:expires].to_i.should be_within(1).of(1.hour.from_now.to_i)
end
```

This technique is a great example of Test Spy described in Gerard Meszaros book xUnit Test Patterns. Basically, you install a spy and check the results collected by the test spy in the verification phase. In this case the Hash is the Test Spy that collects data. See how the stub is used to install the spy in the SUT? It overcomes the problems and isolates the SUT from the Rails framework and allows the code to be tested easily. 

In my TDD bootcamps, the topic on Stubs and Mocks generates lot of discussion. To clear confusion that surrounds the stubs and mocks, I would state : Read Martin Fowler's paper on Mocks Aren't Stubs. Stub can never fail your test, only mocks can fail your test. Using stubs in combination with a spy like this makes stubs seem like they can in fact fail your test. But only the data collected by the Test Spy decides whether the test passes or not. So the stub's main purpose is to just isolate the production code from Rails framework and allow access to the internal state of the SUT where there is no direct way to access it.
