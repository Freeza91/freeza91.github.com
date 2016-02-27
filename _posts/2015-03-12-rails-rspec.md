---
layout: post
title: rails rspec 
category : [rails, learn, rspec]
tags : [rails, learn, rspec]

---
Rails Rspec 

## rails gem
	group :development, :test do
	  gem 'rspec-rails'
	  gem 'factory_girl_rails'
	end

	group :test do
	  gem "faker"
	  gem "capybara"
	  gem "database_cleaner"
	  gem "launchy"
	  gem "selenium-webdriver"
	end

## rspec install 
	rails g rspec:install 
	
> 生成spec/rails_helper.rb和rails/spec_helper.rb 

#### 常用到的rspec的一些方法
	except(:model or string).to or not_to(是和不是)
	be_valid
	include 
	eq
	
	
	
	
	
	
	
	
	
