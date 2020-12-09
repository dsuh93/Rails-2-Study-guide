### Rails Testing: Setup
- Gemfile:
    - groups:
        - development: testing in localhost
        - test: testing application in realworld
    - `group :test do`
        - gem `'rspec-rails'`: lets us write rspec code in rails
        - gem `'capybara'`: tests our views
        - gem `'shoulda-matchers'`: detailed error messages and nice oneline code for validations
        - gem 'launchy': lets us use launch external apps from within rails
    - `group :development, :test do`
        - gem 'factory_bot_rails': lets us auto create instances of models while we're testing, let's us create many instances
        - gem 'faker': random instance generator
    
### next step
1. bundle install
2. rails g rspec:install

3. .rspec (file)
    - --require spec_helper
    - --color
    - --format documentation
4. /application.rb
```Ruby
#under config.active_record.raise_in_transactional_callbacks = true
    config.generators do |g|
        g.test_framework :rspec,
            :fixtures => false,
            :view_specs => false,
            :helper_specs => false,
            :routing_specs => false,
            :controller_specs => true,
            :request_specs => false
        g.fixture_replacement :factory_bot, :dir => "spec/factories"
    end
#it will automatically create testing files with code in it
```
5. rails g model TestModel
    - in spec/models/test_model_spec.rb
```Ruby
    require 'rails_helper'

    RSpec.describe TestModel, type: :model do
        pending "add some examples to (or delete) #{__FILE__}"
    end
```

### Rails testing: Models
- unit tests for models:
    - spec/models/capy_spec.rb
    - what do we test in the model?
        - validations
        - associations
        - class methods
        - error messages
    1. describe/it blocks
```Ruby
    describe "validations" do
        # it "should validate presence of name" do
        #     capy = Capy.new(color: 'blue')
        #     expect(capy.valid?).to be false
        # end

        it { should validate_presence_of(:name) } #thanks to shoulda-matcher gem
        it { should validate_presence_of(:color) }
        it { should validate_uniqueness_of(:name) }
        # it "should validate presence of color" 
        # it "should validate uniqueness of name"
        it "should validate color is not green" do 
            green_capy = Capy.new(name: "deb", color: "green")
            green_capy.valid?
            expect(green_capy.errors[:color]).to eq(["cannot be green"])
        end
    end

    describe "associations" do 
        # it "should have many parties"
        it { should have_many(:parties)}
        it { should have_many(:attendances)}
        it { should have_many(:parties_to_attend).thorugh(:attendance)}
        it "should have many attendances"
        it "should have many parties to attend"
    end

    describe "class methods" do
        describe "::capys_of_the_rainbow" do
            it "should return all capys of color rainbow" do
                # rainbow = Capy.create(name: "rainbow", color: "rainbow")
                # not_rainbow = Capy.create(name: "not a rainbow", color: "sparkles")
                # expect(Capy.capys_of_the_rainbow).to include(rainbow)
                # expect(Capy.capys_of_the_rainbow).not_to include(not_rainbow)
                expect(Capy.capys_of_the_rainbow.where_values_hash).to eq("color" => "rainbow")
            end
        end
    end
```

### Rails Testing: Factories
- use factory-bot and faker to make life easier
    - factory-bot creates instances for testing:
        - spec/factories/capy_model
```Ruby
    FactoryBot.define do
        factory :capy do
            name { Faker::Name.name } # {} ensure randomness
            color { Faker::Color.hex_color }

            factory :green_capy do
                color "green"
            end
        end
    end
```
- back in spec/models/capy_spec.rb
```Ruby
    subject(:capy) {FactoryBot.build(:capy)}

    #can overwrite specific attributes
    green_capy = FactoryBot.build(:green_capy)
    green_capy = FactoryBot.create(:green_capy) #use this for saving to db and retrieval later
```

### Rails Testing: Controllers
- What to test?
    - status codes and reponses in various conditions
    - valid and invalid params
- rspec rails api
    - navigation
        - get, post, patch, delete
    - assertions
        -render_template
        - redirect_to
        - have_http_status, be_success
- example:
```Ruby
    RSpec.describe DemoController, type: :controller do
        describe "GET #index" do
            it "responds successfully with an HTTP 200 status code" do
            get :index
                expect(response).to have_http_status(200)
                expect(response).to render_template(:index)
            end
        end
    end
```
```Ruby
# each controller action will have a spec
# spec/controllers/capy_controller_spec.rb

require 'rails_helper'
RSpec.describe CapysController, type: :controller do
    describe "GET #index" do
        it "renders the capy's index"
            get :index
            expect(response).to be_success
            expect(response).to render_template(:index)
    end

    describe "GET #show" do
        it "renders the show template" do
            Capy.create!(name: "Deb", color: "blue")
            get :show, id: 1
            expect(response).to render_template(:show)
        end

        context "if the capybara does not exist" do
            it "is not a success" do
                begin
                    get :show id: -1
                rescue
                    ActiveRecord::RecordNotFound
                end

                expect(response).not_to render_template(:show)
            end
        end
    end

    describe "POST #create" do
        context "with invalid params" do
            it "renders the new template" do
            post :create, capy: { name: "Kath"}
            expect(response).to render_template(:new)
            end
        end

        context "with valid params" do
            it "redirects to capy page on success" do
            post :create, capy: {name: "Kath", color: "maroon" }
            expect(response).to redirect_to(capy_url(Capy.find_by(name: "Kath")))
        end
    end
end
```

### Rails Testing: Features, End to End, Integration
- capybara: testing rails features
- capybara api
    - navigation
        - visit
        - click_on
    - forms
        - fill_in "field", with: "value"
        - choose, check, uncheck
        - select "option", from: "select box"
    - assertions
        - have_content
        - current_path
        - page
    - debugging
        - save_and_open_page
```Ruby
    #spec/features/capy_features_spec.rb

require 'rails_helper' #this requires spec_helper

feature "capybara features", type: :feature do
    feature "making a new capybara" do
        # before(:each) do 
        #     visit "/capys/new"
        # end
        # ^ don't need this cause of spec helper

        scenario "with invalid params" do
            # fill_in "name", with: "pam"
            # click_on "Create Capy"
            creat_capy("pam", nil) #spec_helper
            expect(current_path).to eq("/capys")
            expect(page).to have_content("Color can't be blank")
        end

        scenario "with valid params" do
            # fill_in "name", with: "pam"
            # fill_in "color", with: "pink"
            # click_on "Create Capy"
            # save_and_open_page can help you if you forget where you're supposed to end up
            creat_capy("pam", "pink") #spec_helper
            expect(page).to have_content("pam")
            expect(current_path).to eq("/capys/#{Capy.find_by(name: "pam").id}")
        end
    end
end

#in spec_helper
def create_capy(name, color)
    visit "/capys/new"
    fill_in "name", with: name
    fill_in "color", with: color
    click_on "Create Capy"
end
```

### Rails testing: Views
- View spec are marked by :type => :view
- use them to test the content of view templates
    - without invoking a specific controller.

- Three general steps:
    1. Use the `assign` method to set ivars in the view.
    2. Use the `render` method to render the view.
    3. Set expectations against the resulting rendered template.
- looking at a simple index view:
```HTML
<h1>Cat App</h1>
<div>
  <ul>
    <% @cats.each do |cat| %>
    <li>
      <a href="<%= cat_url(cat) %>"><%= cat.name %></a>
    </li>
    <% end %>
  </ul>
</div>
```
- before that make sure you install rspec properly:
    - add rspec-rails to both the development and test groups of gemfile
    - then in project directory
        - bundle install
        - rails generate rspec:install
    - can create boilerplate specs with rails generate
        - rails generate rspec:model user

- in spec/views/cats/index.html.erb_spec.rb
```Ruby
require "rails_helper"

RSpec.describe "cats/index" do
    let(:jet) { Cat.create!(name: "Jet")}
    let(:frey) { Cat.create!(name: "Frey")}
    it "render Cat names" do
        # here we're assigning the index template's ivar
        # value of @cats
        assign(:cats, [jet, frey])
        # this method will render the template with the
        # above assigned ivar
        render
        # we can check the "rendred" output for a match
        # in the HTML
        expect(rendered).to match /Jet/
        expect(rendered).to mathc /Frey/
    end

    it "has a header with the name of the application" do
        assign(:cats, [jet])
        render
        expect(rendered).to match /Cat App/
    end

    it "has a link to each cat's show page" do
        assign(:cats, [jet])
        render
        expect(rendered).to have_link 'Jet', href: cat_url(jet.id)
    end
end
```