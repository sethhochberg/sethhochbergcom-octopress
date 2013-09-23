---
layout: post
title: "Storyward"
date: 2013-09-20 17:21
comments: true
categories: web code 
---

Storyward is a web application for collaborative storybuilding in the
style of a create-your-own-advendute book, built with Ruby on Rails in a
7-day sprint at Dev Bootcamp in San Francisco. 

{% img http://media.sethhochberg.com/storyward_home.png %}

Storyward is neat because it was a great opprotunity to use a classical
data structure, the simple tree, at a relativley high level. All of the
possible paths through a story are built out of collections of related "nodes", a "story" is simply a single path through the node tree. 

Having a hard time visualizing that based on the description? 

My favorite part of this project was finding the most intuitive
way to represent this scheme for story data was to show the data
structure itself to the user.

{% img http://media.sethhochberg.com/storyward_tree.png %}

We store stories in the database as a tree, and then represent the data
to the user in the exact same way. These tree plots are built using the
D3 data visualization library, and are fully interactive. A user can
grab a node, drag it to a better position for them to view, double-click
to close or expand a branch of the tree for easier browsing, etc. At all
times the user sees a path (traced above in blue) representing the current
story path they are reading, and can hover over any node to preview that
node's content, making a simple and intuitive process for a user to
explore the different branches in a story which they can read.

Tree-like data structures are classically built and accessed using
recursive algorithms, but, this wasn't an ideal way to operate for us.
Since our individual nodes was its own database row, accessing a path of
nodes through a tree recursivley, only fetching direct parents or
children, would become a very expensive process - 
making many single-record database queries to represent any story (or
possibly many hundreds of single queries to represent an entire tree). 

The solution to this problem was implementing some cache-like metadata
on each node, making it aware of the entire parent path through the tree
to that particular point.

{% codeblock Node Schema lang:ruby %}
  create_table "nodes", force: true do |t|
      t.integer "user_id"
      t.string "title"
      t.text "content"
      t.integer "parent_node"
      t.integer "parent_path", default: [], array: true
      t.integer "children_nodes", default: [], array: true
      t.boolean "terminal"
  end
{% endcodeblock %}

{% codeblock Node Model Methods lang:ruby %}
  before_create :build_parent_path

  def parent_chain
    find_ordered_from_array(self.parent_path)
  end

  def children_objects
    find_ordered_from_array(self.children_nodes)
  end  

  private
  def build_parent_path
    unless self.parent_node == 0
      self.parent_path = self.class.find(parent_node).parent_path
      self.parent_path.push(parent_node)
      self.parent_path_will_change!
    end
  end

  def find_ordered_from_array(ids_array)
   unordered_nodes = self.class.find(ids_array)
    ids_array.map{|id| unordered_nodes.detect{|each| each.id == id}}
  end
{% endcodeblock %}

The call to "self.parent_path_will_change!" is interesting -
ActiveRecord does not track destructive changes to objects, including
pushing to or popping from arrays. We need to explicitly tell
ActiveRecord to expect that the parent_path will change in order for
the changed attribute to be persisted in the database. 
