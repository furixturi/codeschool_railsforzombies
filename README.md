# Rails for Zombies Play through
Notes from the code school workshop [rails for zombies](http://railsforzombies.org/)

## Level 1 DEEP IN THE CRUD

### Syntax

* Hash

	```ruby
	b = {
		id: 3,
		status: "I ate some brains",
		zombie: "Tim"
	}
	```

	* Accessing value with keys using dot syntax

		```ruby
		b.id
			=> 3
		```

		or the alternative hash (`[:key]`) syntax

		```ruby
		b[:id]
			=> 3
		```

* (useful) print string

	```ruby
	puts b[:id]
	```

### Dealing with the database:

The zombie database structure:

#### tweets

id | status | zombie
---|--------|-------
1  | Where can I get a good bite to eat? | Ash
2  | My left arm is missing, but I don't care | Bob
3  | I just ate some delicious brains | Jim
4  | OMG, my fingers turned green. #FML | Ash 

#### I. CREATE

Three ways to create a new zombie tweet entry:

1. verbose way

	```ruby 
	t = Tweet.new
	t.status = "I <3 brains."
	t.save
	```

2. less verbose way

	```ruby
	t = Tweet.new(
		status: "I <3 brains.",
		zombie: "Jim"
	)
	t.save
	```

3. compact way

	```ruby
	Tweet.create(
		status: "I <3 brains.",
		zombie: "Jim"
	)
	```

#### II. READ

1. Find

	1. find by id

		```ruby
		Tweet.find(2)
		```

	2. find multiple by id

		```ruby
		Tweet.find(2, 3, 4)
		```

	3. find the first, the last, find all

		```ruby
		Tweet.first
		```

		```ruby
		Tweet.last
		```

		```ruby
		Tweet.all
		```

2. count the number of entries

	```ruby
	Tweet.count
	```

3. order entries by a certain key value

	```ruby
	Tweet.order(:zombie)
	```

4. return the first 10 entries

	```ruby
	Tweet.limit(10)
	```

5. find all entries with a certain key-value pair

	```ruby
	Tweet.where(zombie: "ash")
	```

6. chaining them together
	
	* To find the first zombie tweet by the zombie "ash":

	```ruby
	Tweet.where(zombie: "ash").first
	```

	* To find Ash's first ten tweet ordered by status:

	```ruby
	Tweet.where(zombie: "ash").order(:status).limit(10)
	```

#### III. UPDATE

1. verbose way

	```ruby
	t = Tweet.find(3)
	t.zombie = "EyeballChomper"
	t.save
	```

2. less verbose way

	```ruby
	t = Tweet.find(2)
	t.attributes = {
		status: "Can I munch your eyeballs?",
		zombie: "EyeballChomper"
	}
	```

3. compact way

	```ruby
	t = Tweet.find(2)
	t.update(
		status: "Can I munch your eyeballs?",
		zombie: "EyeballChomper"
	)
	```

#### IV. DELETE

1. delete one

	```ruby
	t = Tweet.find(2)
	t.destroy
	```

2. delete all

	```ruby
	Tweet.destroy_all
	```

## Level 2 MODELS TASTE LIKE CHICKEN

### Model 
is the way how your Rails application communicates with a data store.

1. To define a Model class

	```ruby
	class Tweet < ActiveRecord::Base

	end
	```

	`<` is for `extends`

	This `Tweet` class then maps to the `tweets` table

	* tweets

		id | status | zombie
		---|--------|--------
		1  | Where can I get a good bite to | Ash
		2  | My left arm is missing, but I  | Bob
		3  | I just ate some delicious      | Jim
		4  | OMG, my fingers turned green.  | Ash


2. Validations

	We don't want empty records to be saved in the database, so we add validation code in our model class:

	```ruby
	class Tweet < ActiveRecord::Base
		validates_presence_of :status
	end
	```

	* Now trying to save an empty record will give you an error

		```
		>> t = Tweet.new
			=> #<Tweet id: nil, status: nil, zombie: nil>
		>> t.save
			=> false
		>> t.errors.messages
			=> {status: ["can't be blank"]}
		>> t.errors[:status][0]
			=> "can't be blank"
		```	

	* Other validation possibilities:
		* presence, numeric, unique:

			```ruby
			validates_presence_of :status
			validates_numericality_of :fingers
			validates_uniqueness_of :toothmarks
			```
		
		* special pattern: confirmation pairs
		
			```ruby		
			validates_confirmation_of :password		# to validate if two input fields match
													# (one of which is a confirmation field 
													# like in the case of a password or email) 
			```

			An example usage

			in `Model`:

			```ruby
			class Person < ActiveRecord::Base
				validates_confirmation_of :password, message: "should match"
				validates_presence_of :password_confirmation, if: :password_changed?
			end
			```

			in `View`:

			```ruby
			<%= password_field "person", "password" %>
			<%= password_field "person", "password_confirmation" %>
			```

		* requires a checkbox to be checked

			```ruby
			validates_acceptance_of :zombification	# to validate if a checkbox is checked
			```

		* validate length

			```ruby
			validates_length_of :password, minimum: 3
			```

		* validates format

			```ruby	
			validates_format_of :email, with: /regex/i  # provide a regular expression 
														# to validate email
			```

		* validates number range
			```ruby
			validates_inclusion_of :age, in: 21..99
			validates_exclusion_of :age, in: 0...21, 
				message: "Sorry, you must be over 21"
			```

	* Compact alternate syntax of validation

		```ruby
		validates :status, 
					presence: true, 
					length: { minimum: 3 }
		```

		Additional options

		```ruby
				presence: true
				uniqueness: true
				numericality: true
				length: { minimum: 0, maximum: 2000 }
				format: { with: /.*/ }
				acceptance: true
				confirmation: true
		```

### Relationships

1. Relational Database

	We have the tweets table, now we store zombies in their own table with unique ids and graveyard information. The two tables now have a _relationship_:

	> each zombie `has_many` tweets

	and

	> each tweet `belongs_to` a zombie

	Now the zombies show up in the tweets table with their unique id *zombie_id*, with which we can find the very zombie in the zombies table.

	* zombies

		id | name | graveyard
		---|------|-----------
		1 | Ash | Glen Haven Memorial Cemetery
		2 | Bob | Chapel Hill Cemetery
		3 | Jim | My Father's Basement

		```ruby
		class Zombie < ActiveRecord::Base
			has_many :tweets	# plural!
		end
		```

	* tweets

		id | status | zombie_id
		---|--------|-----------
		1 | Where can I get a good bite to eat? | 1
		2 | My left arm is missing, but I don't care | 2
		3 | I ust ate some delicious brains. | 3
		4 | OMG, my fingers turned green. #FML | 1

		```ruby
		class Tweet < ActiveRecord::Base
			belongs_to :zombie 		# singular!
		end
		```

2. Using relationships

	* Pass a zombie when creating a new tweet
 
		```ruby
		> ash = Zombie.find(1)
			=> #<Zombie id: 1, name: "Ash", graveyard: "Glen Haven Memorial Cemetery">
		> ash.tweets.count
			=> 2
		> t = Tweet.create(status: "Your eyelids taste like bacon.", zombie: ash)
			=> #<Tweet id: 5, status: "Your eyelids taste like bacon.", zombie_id: 1>
		> ash.tweets
			=> [
				#<Tweet id: 1, status "Where can I get a good bite to eat?", zombie_id: 1>,
				#<Tweet id: 4, status "OMG, my fingers turned green. #FML", zombie_id: 1>,
				#<Tweet id: 5, status "Your eyelids taste like bacon.", zombie_id: 1>
			]
		```
	* Get the zombie from an existing tweet

		```ruby
		> t = Tweet.find(5)
			=> #<Tweet id: 5, status: "Your eyelid tastes like bacon.", zombie_id: 1>
		> t.zombie
			=> #<Zombie id: 1, name: "Ash", graveyard: "Glen Haven Memorial Cemetery">
		> t.zombie.name
			=> "Ash"
		```

## Level 3 THE VIEWS AIN'T ALWAYS PRETTY

## Level 4 CONTROLLERS MUST BE EATEN












