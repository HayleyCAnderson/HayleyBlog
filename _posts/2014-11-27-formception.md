---
layout: post
title: Formception
---

Or how to make forms within forms to make forms within forms. The key here is the magic (yes, actual magic) of `accepts_nested_attributes_for`.

When we recently had the opportunity to choose a group project to work on, I insisted on the [Survey Monkey clone](http://formr-orlatter.herokuapp.com) because I thought it would present some interesting challenges. Did I actually have any idea of how to handle those challenges? No, not really, but it was quite a fun journey.

We decided pretty quickly to have separate tables and models for surveys, questions, and answers, but have only a one-page form for creating a survey complete with questions and answers. This then posed the problem of how to best create the questions and answers with a form for a survey.

As it turns out, `accepts_nested_attributes_for` is a great way to solve this problem. It must be added to each model that will be creating objects of another model within it - in this case, survey accepts nested attributes for questions and answers, and question accepts nested attributes for answers.

Then the magic happens. Create the correct number of new empty questions and answers in the surveys controller, create a form for survey (just survey!), add a `fields_for :questions`, and looped under that, a `fields_for :answers`. The numbers of questions and answers created will be rendered by the mini-forms `fields_for` and then given ids. Since question belongs to survey, the questions table will receive the survey id, and since answer belongs to question, the answers table will receive the question id.

It gets better. Add `allow_destroy: true`, and you can delete questions and answers directly within the survey form by including an option for `:_destroy` in each `fields_for`. Add `reject_if: :all_blank`, and the object will not be created if the field is not filled out. Questions, since they may have empty answers within them, need only a slightly more detailed lambda stating to reject if the main question field is blank.

{% highlight ruby %}
# Check out this crazy model magic

class Survey < ActiveRecord::Base
  validates :name, presence: true

  belongs_to :user
  default_scope { order(id: :desc) }

  has_many :questions, dependent: :destroy
  has_many :answers, through: :questions

  accepts_nested_attributes_for(
    :questions,
    allow_destroy: true,
    reject_if: lambda { |attributes| attributes["content"].blank? }
  )
  accepts_nested_attributes_for(
    :answers,
    allow_destroy: true,
    reject_if: :all_blank
  )
end
{% endhighlight %}

Now the models and form are set, but there's still a bit more work to be done. The survey params now contain _everything_: `:name, questions_attributes: [:content, :_destroy, answers_attributes: [:choice, :_destroy]]`. And since the form creates ids for questions and answers, it will continue creating new ids each time you try to edit unless you specifically pull in the ids for questions and answers through the params in the surveys controller.

And that's the first half of your formception. Here, Rails has it all set up for you.
