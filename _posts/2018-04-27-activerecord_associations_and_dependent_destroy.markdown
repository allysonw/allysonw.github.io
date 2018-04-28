---
layout: post
title:      "ActiveRecord Associations and dependent: :destroy"
date:       2018-04-27 21:28:17 -0400
permalink:  activerecord_associations_and_dependent_destroy
---


While developing Dental Office Manager I came across a problem when deleting records from models whose primary key was used as a foreign key for another model. For example, in my app a Patient `has_many` appointments, and a User also `has_many` appointments. Appointments is the join model between Users and Patients.

<img src="https://i.imgur.com/cW6Z15j.png" alt="ERD" style="width: 600px;"/>


When a site visitor decided to delete a Patient from the app, all went smoothly on the surface. But then, when they then went to display the Appointments index page, which included a table listing each Appointment in the DB and the associated Patient, my app threw a no method error. For each Appointment whose associated Patient had been deleted, the app was unable to print the Patient's name who had previously been associated to that appointment. That made sense because, well, the Patient had been deleted from the DB!

I considered my options. One option was to implement logic such that I would not attempt to display a Patient's name for an Appointment that no longer had a Patient associated. This might make sense in certain domain models, but in my app it didn't. If a Patient is no longer in my database, how can that Patient still have standing Appointments in the app?

I decided that I needed to, upon destruction of a Patient, also destroy the patient's associated Appointments, leaving no orphaned records in my Appointments table. As clearly explained [here](http://guides.rubyonrails.org/association_basics.html#why-associations-questionmark), all this required was adding the `dependent: :destroy` option to the model that implements the `has_many` relationship, in my case, the Patient model. I also ended up adding it to my User model to ensure the same behavior upon deleting a User.

```
# app/models/patient.rb
class Patient < ActiveRecord::Base
  has_many :appointments
  has_many :users, through: :appointments, dependent: :destroy
end
```

```
# app/models/user.rb
class User < ActiveRecord::Base
  has_many :appointments
  has_many :patients, through: :appointments, dependent: :destroy
end
```

```
# app/models/appointment.rb
# Join Model
class Appointment < ActiveRecord::Base
  belongs_to :user
  belongs_to :patient
end
```

Now, whenever a Patient or a User model is destroyed, all associated Appointment records are too.

Interestingly, I declared `dependent: :destroy` on the `has_many` statement indicating that appointments was my join table. Using `dependent: destroy` on the `has_many :appointments` statement did not achieve the desired behavior. I'm not quite sure why...

To sum up: `dependent: :destroy` should be used whenever you need to destroy all child associated models when the parent model is destroyed.

