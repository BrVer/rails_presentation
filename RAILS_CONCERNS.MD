http://artemeff.com/2013/04/21/concerns-v-rails-4.html
http://habrahabr.ru/post/125822/
http://engineering.appfolio.com/2013/06/17/ruby-mixins-activesupportconcern/

итак, типичная модель:

```ruby
class User < ActiveRecord::Base

  # class code
  
  attr_accessor :foo

  has_many :authentications, :dependent => :destroy
  has_many :invitees, :class_name => 'User', :as => :invited_by

  delegate :city, :country, :to => :location

  attr_accessible :admin, :banned, :as => :admin

  mount_uploader :userpic, UserPicUploader

  scope :admin, where(:admin => true)

  validates :username,
            :uniqueness => true,
            :format => /^[a-z][a-z\d_-]+$/,
            :length => { :within => 3..20 },
            :exclusion => { :in => USERNAME_EXCLUSION }

  # instance methods
  
  def name
    [ first_name, last_name ].filter(&:presence).compact.map(&:strip) * ' '
  end
  
  # class methods
  
  def self.find_old
    where('created_at < ?', 1.year.ago).all
  end
    
end
```


как это DRYить с помощью модулей:

```ruby

# user_stuff.rb
module UserStuff

  def self.included(base)
    # экстендим модель методами класса
    base.extend ClassMethods

    # инклудим методы экземпляра
    # Module#include — приватный метод. Его по задумке можно вызывать только внутри определения класса
    # Поэтому мы не можем написать base.include(InstanceMethods), а приходится делать так:
    base.send :include, InstanceMethods

    # а дальше пошли валидации и прочее # вот тут вопрос instance_eval или class_eval
    base.instance_eval do
      validates :gender, :presence => true
    end
    
  end


  # class methods
  module ClassMethods
    def find_old
      where('created_at < ?', 1.year.ago).all
    end
  end

  # instance methods
  module InstanceMethods
    def name
      [ first_name, last_name ].filter(&:presence).compact.map(&:strip) * ' '
    end
  end

end

# user.rb
class User < ActiveRecord::Base
  include UserStuff
end

```


концерн:

```ruby
module UserStuff
  extend ActiveSupport::Concern

  included do
    validates :gender, :presence => true
  end

  # class methods
  class_methods do
    def find_old
      where('created_at < ?', 1.year.ago).all
    end
  end

  # instance methods
  instance_methods do
    def name
      [ first_name, last_name ].filter(&:presence).compact.map(&:strip) * ' '
    end
  end

end
```