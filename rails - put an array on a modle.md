
Problem:  You want to have something like


    u = User.first
    u.courses
    => [ "Yatting", "Fencing", "Drag Racing", "Properly Taught Statistics", "Ruby Reloaded!" ]


Well, how do you do that?  Let's create an example app!


Solution:

You need to store a new ActiveRecord model collection on your currently existing User ActiveRecord model.  Then in your database seeding script, you can set the proper courses of each user as they are created, OR make it so they can add courses to their own model.  

1)  Create the models and things
2)  Fill in the associations on the model scripts
3)  Fill in the magic, so we can address courses as though it's just an array of names!


1)  Create the models and things


    $  rails new model_collection_test
    $  cd model_collection_test

    $  rails generate scaffold user
    $  rails generate model course name:string user_id:integer
    $  rails generate model users_courses user_id:integer course_id:integer




2)  Fill in the associations on the model scripts

#####(app/models/course.rb)#####
    class Course < ActiveRecord::Base
      belongs_to :user
    end


#####(app/models/user.rb)#####
    class User < ActiveRecord::Base
      has_many :courses
    end



3)  Now We need to make it so we can pull an array of course names out of the user model.  

Change the user model to look like this:

#####(app/models/user.rb)#####
    class User < ActiveRecord::Base
      has_many :courses
      
      # We need this function which will convert all 
      # the ActiveRecord "course" objects owned by this user into an array of 
      # these course's names.  (eew... what an ugly English sentence =/).  
      def enrollment
        courses = self.courses
        name_array = courses.map { |course| course.name }
        
        return name_array
      end
    end


Ok, see there?  We couldn't just name it plain old "courses" because that name is taken by the ActiveRecord name "course.rb" which translates to "Courses" when you ascribe the has_many relationship.  But this is still pretty cool, right?  We just used a basic getter.  Now let's play with it in the console!  Afterwards we can impliment a setter method!!!


    $  rails console

    Loading development environment (Rails 3.0.9)

    > u = User.create
     => #<User id: 3, created_at: "2012-08-12 15:10:20", updated_at: "2012-08-12 15:10:20">

    > u.courses.create(:name => "Ruby Reloaded!")
     => #<Course id: 4, created_at: "2012-08-12 15:10:38", updated_at: "2012-08-12 15:10:38", user_id: 3, courses: nil, name: "Ruby Reloaded!">

    > u.courses
     => [#<Course id: 4, created_at: "2012-08-12 15:10:38", updated_at: "2012-08-12 15:10:38", user_id: 3, courses: nil, name: "Ruby Reloaded!">]

    > u.enrollment
     => ["Ruby Reloaded!"]




So now let's impliment a setter so we can create these special records with out having to think about active record at all =D

#####(app/models/user.rb)#####
    class User < ActiveRecord::Base
      has_many :courses
      
      def enrollment
        courses = self.courses
        name_array = courses.map { |course| course.name }
        
        return name_array
      end
      
      def enrollment=(val)
        self.courses.delete_all
        self.courses.create!(:name => val)
      end
    end




Now we want to be able to push more courses onto a user object.  

#####(app/models/user.rb)#####
<pre>
    class User < ActiveRecord::Base
      has_many :courses
      
      def enrollment
        courses = self.courses
        name_array = courses.map { |course| course.name }
        
        return name_array
      end
      
      def enrollment=(val)
        self.courses.delete_all
        self.courses.create!(:name => val)
      end
      
      # FAIL
      def enrollment+=(val)
        puts "hello"
      end
    end
</pre>

