.. _diagrams:

========
Diagramy
========

.. graphviz::

   digraph foo {
      "bar" -> "baz";
   }


.. graphviz::

    digraph Flatland {

      a -> b -> c -> g;
      a  [shape=polygon,sides=4]
      b  [shape=polygon,sides=5]
      c  [shape=polygon,sides=6]

      g [peripheries=3,color=yellow];
      s [shape=invtriangle,peripheries=1,color=red,style=filled];
      w  [shape=triangle,peripheries=1,color=blue,style=filled];

    }


.. uml::

      @startuml

      'style options
      skinparam monochrome true
      skinparam circledCharacterRadius 0
      skinparam circledCharacterFontSize 0
      skinparam classAttributeIconSize 0
      hide empty members

      Class01 <|-- Class02
      Class03 *-- Class04
      Class05 o-- Class06
      Class07 .. Class08
      Class09 -- Class10

      @enduml


.. uml::

   @startuml

   [*] --> MyState
   MyState --> CompositeState
   MyState --> AnotherCompositeState
   MyState --> WrongState

   CompositeState --> CompositeState : \ this is a loop
   AnotherCompositeState --> [*]
   CompositeState --> [*]

   MyState : this is a string
   MyState : this is another string

   state CompositeState {

   [*] --> StateA : begin something
   StateA --> StateB : from A to B
   StateB --> StateA : from B back to A
   StateB --> [*] : end it

   CompositeState : yet another string
   }

   state AnotherCompositeState {

   [*] --> ConcurrentStateA
   ConcurrentStateA --> ConcurrentStateA

   --

   [*] --> ConcurrentStateB
   ConcurrentStateB --> ConcurrentStateC
   ConcurrentStateC --> ConcurrentStateB
   }

   note left of WrongState
      This state
      is a dead-end
      and shouldn't
      exist.
   end note

   @enduml



.. uml::

   @startuml

   start

   :first activity;

   :second activity
    with a multiline
    and rather long description;

   :another activity;

   note right
     After this activity,
     are two 'if-then-else' examples.
   end note

   if (do optional activity?) then (yes)
      :optional activity;
   else (no)

      if (want to exit?) then (right now!)
         stop
      else (not really)

      endif

   endif

   :third activity;

   note left
     After this activity,
     parallel activities will occur.
   end note

   fork
      :Concurrent activity A;
   fork again
      :Concurrent activity B1;
      :Concurrent activity B2;
   fork again
      :Concurrent activity C;
      fork
      :Nested C1;
      fork again
      :Nested C2;
      end fork
   end fork

   repeat
      :repetitive activity;
   repeat while (again?)

   while (continue?) is (yes, of course)
     :first activity inside the while loop;
     :second activity inside the while loop;
   endwhile (no)

   stop

   @enduml