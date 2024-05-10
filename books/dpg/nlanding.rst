The Airport Application
=======================

In this chapter, we present the entire first draft of our airport
example application. The code in this chapter is complete, and, given
the libraries defined in :doc:`time-mod`, and :doc:`heap`, the code
should run in a standard Dylan implementation. This example pulls
together many of the techniques presented so far.

The ``definitions.dylan`` file
------------------------------

This file contains common definitions that are used throughout several
libraries in the airport example.

The ``definitions.dylan`` file.

.. code-block:: dylan

    module: definitions

    // This file contains constants and other definitions used in common with
    // the other parts of the airport example

    // The capital letters of the alphabet
    define constant $letters = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";

    // This type represents positive integers
    define constant <positive-integer> = limited(<integer>, min: 1);

    define constant $hours-per-day = 24;

    define constant $minutes-per-hour = 60;

    define constant $seconds-per-minute = 60;

    define constant $seconds-per-hour
      = $minutes-per-hour * $seconds-per-minute;

    // This method returns the union of the false type and a type you specify,
    // as a simple shorthand
    // This method may already be provided by your Dylan implementation
    define method false-or (other-type :: <type>) => (combined-type :: <type>)
      type-union(singleton(#f), other-type);
    end method false-or;

The ``airport-classes.dylan`` file
----------------------------------

This file contains all the main classes specific to the airport example.
Several methods that describe or initialize these objects are included
as well.

Physical objects
~~~~~~~~~~~~~~~~

The classes that follow describe fundamental attributes of tangible
objects.

The *airport-classes.dylan* file.

.. code-block:: dylan

    module: airport

    // PHYSICAL OBJECTS AND SIZE

    // Used to keep track of object dimensions and object capacities
    // All dimensions are in feet
    define class <size> (<object>)
      slot length :: <positive-integer>, init-keyword: length:;
      slot width :: <positive-integer>, init-keyword: width:;
      slot height :: <positive-integer>, init-keyword: height:;
    end class <size>;

    define abstract class <physical-object> (<object>)
      slot current-position :: <position>, init-keyword: current-position:;
      slot physical-size :: <size>, init-keyword: physical-size:;
    end class <physical-object>;

    define method say (physical-object :: <physical-object>) => ()
      format-out("object at ");
      say(physical-object.current-position);
    end method say;

In the preceding portion of the ``airport-classes.dylan`` file, we define
the class ``<size>``, which allows us to specify the external dimensions
and container volume of various objects. For example, we might want to
specify that certain gate areas might be too small to hold the large
aircraft. We also define the base class for all tangible objects,
``<physical-object>``.

Next, we define the classes where aircraft are normally located.

.. _nlanding-vehicle-containers:

Vehicle containers
~~~~~~~~~~~~~~~~~~

The ``airport-classes.dylan`` file. *(continued)*

.. code-block:: dylan

    // VEHICLE STORAGE

    // The default size for a vehicle container
    define constant $default-capacity
      = make(<size>, length: 350, width: 200, height: 100);

    // This class represents a location where an aircraft could be stored
    define abstract class <vehicle-storage> (<physical-object>)
      slot storage-capacity :: <size> = $default-capacity,
        init-keyword: capacity:;
      each-subclass slot name-prefix :: <string> = "Storage", setter: #f;
      slot identifier :: <string>, required-init-keyword: id:;
      slot connected-to :: <simple-object-vector>;
    end class <vehicle-storage>;

    // By using the name-prefix each-subclass slot, we share one say method
    // for all vehicle containers
    define method say (storage :: <vehicle-storage>) => ()
      format-out("%s %s", storage.name-prefix, storage.identifier);
    end method say;

    define method object-fits?
        (object :: <physical-object>, container :: <vehicle-storage>)
     => (fits? :: <boolean>)
      let object-size = object.physical-size;
      let container-capacity = container.storage-capacity;
      object-size.length < container-capacity.length
        & object-size.height < container-capacity.height
        & object-size.width < container-capacity.width;
    end method object-fits?;

    // Vehicle storage that can hold only one aircraft regardless of direction
    // Direction in this context is either #"inbound" or #"outbound"
    define abstract class <single-storage> (<vehicle-storage>)
      slot vehicle-currently-occupying :: false-or(<aircraft>) = #f;
    end class <single-storage>;

    // Vehicle storage that can hold multiple aircraft, with distinct queues
    // for each direction
    define abstract class <multiple-storage> (<vehicle-storage>)
      slot vehicles-by-direction :: <object-table> = make(<object-table>);
      slot maxima-by-direction :: <object-table> = make(<object-table>);
      keyword directions:;
      keyword maxima:;
    end class <multiple-storage>;

    // In a real airport, there would be many paths an aircraft could take
    // For our simple airport example, we define only the #"inbound" and
    // #"outbound" paths
    // The directions parameter is a sequence of these aircraft path names
    // Multiple storage containers can limit the number of aircraft that
    // they can hold for each path; this is the maxima parameter
    // This initialize method creates a queue to hold aircraft for each
    // direction, and stores the queue in a table indexed by direction
    // This method also stores the maximum number of aircaft for that
    // direction in a different table
    define method initialize
         (object :: <multiple-storage>, #key directions :: <sequence>,
          maxima :: <sequence>)
      next-method ();
      for (direction in directions,
           maximum in maxima)
        object.vehicles-by-direction[direction] := make(<deque>);
        object.maxima-by-direction[direction] := maximum;
      end for;
    end method initialize;

    // From the preceding basic vehicle containers, we can build specific
    // containers for each aircraft-transition location
    define class <gate> (<single-storage>)
      inherited slot name-prefix, init-value: "Gate";
    end class <gate>;

    // Given a zero-based terminal number, and a one-based gate number, create
    // an return a string with a gate letter and a terminal number in it
    define method generate-gate-id
        (term :: <nonnegative-integer>, gate :: <positive-integer>)
     => (gate-id :: <string>)
      format-to-string("%c%d", $letters[term], gate);
    end method generate-gate-id;

    // Gates-per-terminal is a vector; each element of the vector is the
    // number of gates to create for the terminal at that index
    // Returns a vector of all the gate instances
    define method generate-gates
        (gates-per-terminal :: <vector>, default-gate-capacity :: <size>)
     => (gates :: <vector>)
      let result = make(<vector>, size: reduce1(\+, gates-per-terminal));
      let result-index = 0;
      for (term from 0 below gates-per-terminal.size)
        for (gate from 1 to gates-per-terminal[term])
          result[result-index]
            := make(<gate>, id: generate-gate-id(term, gate),
                    capacity: default-gate-capacity);
          result-index := result-index + 1;
        end for;
      end for;
      result;
    end method generate-gates;

    // This class represents the part of the airspace over a given airport
    define class <sky> (<multiple-storage>)
      // The airport over which this piece of sky is located
      slot airport-below :: <airport>, required-init-keyword: airport:;
      inherited slot name-prefix, init-value: "Sky";
      required keyword inbound-aircraft:;
    end class <sky>;

    // When a sky instance is created, a sequence of inbound aircraft is
    // provided
    // This method initializes the direction slot of the aircraft to
    // #"inbound", and places the aircraft in the inbound queue of the sky
    // instance
    define method initialize
        (sky :: <sky>, #key inbound-aircraft :: <sequence>)
      next-method(sky, directions: #[#"inbound", #"outbound"],
                  maxima: vector(inbound-aircraft.size,
                                 inbound-aircraft.size));
      let inbound-queue = sky.vehicles-by-direction [#"inbound"];
      for (vehicle in inbound-aircraft)
        vehicle.direction := #"inbound";
        push-last(inbound-queue, vehicle);
      end for;
      // Connect the airport to the sky
      sky.airport-below.sky-above := sky;
    end method initialize;

    // This class represents a strip of land where aircraft land and take off
    define class <runway> (<single-storage>)
      inherited slot name-prefix, init-value: "Runway";
    end class <runway>;

    // Taxiways connect runways and gates
    define class <taxiway> (<multiple-storage>)
      inherited slot name-prefix, init-value: "Taxiway";
    end class <taxiway>;

In the preceding portion of the ``airport-classes.dylan`` file, we define
the tangible objects that represent the various normal locations for
aircraft in and around an airport. These locations are known as
containers or vehicle storage. We can connect vehicle-storage instances
to one another to form an airport. Instances of ``<single-storage>`` can
hold only one aircraft at a time, whereas instances of
``<multiple-storage>`` can hold more than one aircraft at a time. Also,
instances of ``<multiple-storage>`` treat inbound aircraft separately from
outbound aircraft. We define the ``object-fits?`` method, which determines
whether a physical object can fit into a container. We also define
methods for creating, initializing, and describing various containers.
Note the use of the ``each-subclass`` slot ``name-prefix``, which permits
one ``say`` method on the ``<vehicle-storage>`` class to cover all the
vehicle-container classes. Each subclass of vehicle storage can override
the inherited value of this slot, to ensure that the proper name of the
vehicle storage is used in the description of instances of that
subclass.

The ``<vehicle-storage>``, ``<multiple-storage>``, and ``<single-storage>``
classes are all abstract, because it is not sensible to instantiate
them. They contain partial implementations that they contribute to their
subclasses.

In the ``generate-gates`` method, the ``gates-per-terminal`` parameter is a
vector that contains the count of gates for each terminal. By adding up
all the elements of that vector with ``reduce1``, we can compute the
total number of gates at the airport, and thus the size of the vector
that can hold all the gates.

Next, we examine the classes, initialization methods, and ``say`` methods
for the vehicles in the application.

Vehicles
~~~~~~~~

The ``airport-classes.dylan`` file. *(continued)*

.. code-block:: dylan

    // VEHICLES

    // The class that represents all self-propelled devices
    define abstract class <vehicle> (<physical-object>)
      // Every vehicle has a unique identification code
      slot vehicle-id :: <string>, required-init-keyword: id:;
      // The normal operating speed of this class of vehicle in miles per hour
      each-subclass slot cruising-speed :: <positive-integer>;
      // Allow individual differences in the size of particular aircraft,
      // while providing a suitable default for each class of aircraft
      each-subclass slot standard-size :: <size>;
    end class <vehicle>;

    define method initialize (vehicle :: <vehicle>, #key)
      next-method();
      unless (slot-initialized?(vehicle, physical-size))
        vehicle.physical-size := vehicle.standard-size;
      end unless;
    end method initialize;

    define method say (object :: <vehicle>) => ()
      format-out("Vehicle %s", object.vehicle-id);
    end method say;

    // This class represents companies that fly commercial aircraft
    define class <airline> (<object>)
      slot name :: <string>, required-init-keyword: name:;
      slot code :: <string>, required-init-keyword: code:;
    end class <airline>;

    define method say (object :: <airline>) => ()
      format-out("Airline %s", object.name);
    end method say;

    // This class represents a regularly scheduled trip for a commercial
    // airline
    define class <flight> (<object>)
      slot airline :: <airline>, required-init-keyword: airline:;
      slot number :: <nonnegative-integer>,
        required-init-keyword: number:;
    end class <flight>;

    define method say (object :: <flight>) => ()
      format-out("Flight %s %d", object.airline.code, object.number);
    end method say;

    // This class represents vehicles that normally fly for a portion of
    // their trip
    define abstract class <aircraft> (<vehicle>)
      slot altitude :: <integer>, init-keyword: altitude:;
      // Direction here is either #"inbound" or #"outbound"
      slot direction :: <symbol>;
      // The next step this aircraft might be able to make
      slot next-transition :: <aircraft-transition>,
        required-init-keyword: transition:, setter: #f;
    end class <aircraft>;

    define method initialize (vehicle :: <aircraft>, #key)
      next-method();
      // There is a one-to-one correspondence between aircraft instances and
      // transition instances
      // An aircraft can only make one transition at a time
      // Connect the aircraft to its transition
      vehicle.next-transition.transition-aircraft := vehicle;
    end method initialize;

    // The next step an aircraft might be able to make
    define class <aircraft-transition> (<object>)
      slot transition-aircraft :: <aircraft>, init-keyword: aircraft:;
      slot from-container :: <vehicle-storage>, init-keyword: from:;
      slot to-container :: <vehicle-storage>, init-keyword: to:;
      // The earliest possible time that the transition could take place
      slot earliest-arrival :: <time-of-day>, init-keyword: arrival:;
      // Has this transition already been entered in the sorted sequence?
      // This flag saves searching the sorted sequence
      slot pending? :: <boolean> = #f, init-keyword: pending?:;
    end class <aircraft-transition>;

    // Describes one step of an aircraft’s movements
    define method say (transition :: <aircraft-transition>) => ()
      say(transition.earliest-arrival);
      format-out(": ");
      say(transition.transition-aircraft);
      format-out(" at ");
      say(transition.to-container);
    end method say;

    // Commercial aircraft are aircraft that may have a flight
    // assigned to them
    define abstract class <commercial-aircraft> (<aircraft>)
      slot aircraft-flight :: false-or(<flight>) = #f, init-keyword: flight:;
    end class <commercial-aircraft>;

    define method say (object :: <commercial-aircraft>) => ()
      let flight = object.aircraft-flight;
      if (flight)
        say(flight);
      else
        format-out("Unscheduled Aircraft %s", object.vehicle-id);
      end if;
    end method say;

    // The class that represents all commericial Boeing 707 aircraft
    define class <B707> (<commercial-aircraft>)
      inherited slot cruising-speed, init-value: 368;
      inherited slot standard-size,
         init-value: make(<size>, length: 153, width: 146, height: 42);
    end class <B707>;

    define method say (aircraft :: <B707>) => ()
      if (aircraft.aircraft-flight)
        next-method();
      else
        format-out("Unscheduled B707 %s", aircraft.vehicle-id);
      end if;
    end method say;

In the preceding code, we model everything from the most general class
of vehicle down to the specific class that represents the Boeing 707. We
also model the transition steps that an aircraft may take as it travels
throughout the airport, and the airlines and flights associated with
commercial aircraft.

Airports
~~~~~~~~

Finally, we present the class that represents the entire airport and
provide the method that briefly describes the airport.

The ``airport-classes.dylan`` file. *(continued)*

.. code-block:: dylan

    // AIRPORTS

    // The class that represents all places where people and aircraft meet
    define class <airport> (<physical-object>)
      // The name of the airport, such as "San Fransisco International Airport"
      slot name :: <string>, init-keyword: name:;
      // The three letter abbreviation, such as "SFO"
      slot code :: <string>, init-keyword: code:;
      // The airspace above the airport
      slot sky-above :: <sky>;
    end class <airport>;

    define method say (airport :: <airport>) => ()
      format-out("Airport %s", airport.code);
    end method say;

The ``vehicle-dynamics.dylan`` file
-----------------------------------

The ``vehicle-dynamics.dylan`` file contains stubs for calculations that
predict the behavior of the aircraft involved in the example. True
aeronautical calculations are beyond the scope of this book.

The ``vehicle-dynamics.dylan`` file.

.. code-block:: dylan

    module: airport

    // We do not need to type these constants strongly, because the Dylan
    // compiler will figure them out for us

    define constant $average-b707-brake-speed = 60.0; // Miles per hour

    define constant $feet-per-mile = 5280.0;

    define constant $average-b707-takeoff-speed = 60.0; // Miles per hour

    define constant $takeoff-pause-time = 120; // Seconds

    define constant $average-b707-taxi-speed = 10.0;

    define constant $average-b707-gate-turnaround-time
      = 34 * $seconds-per-minute; // Seconds

    // Computes how long it will take an aircraft to reach an airport
    define method flying-time
        (aircraft :: <aircraft>, destination :: <airport>)
     => (duration :: <time-offset>)
      // A simplistic calculation that assumes that the aircraft will
      // average a particular cruising speed for the trip
      make(<time-offset>,
           total-seconds:
             ceiling/(distance-3d(aircraft, destination),
                      aircraft.cruising-speed
                        / as(<single-float>, $seconds-per-hour)));
    end method flying-time;

    // Computes the distance between an aircraft and an airport,
    // taking into account the altitude of the aircraft
    // Assumes the altitude of the aircraft is the height
    // above the ground level of the airport
    define method distance-3d
        (aircraft :: <aircraft>, destination :: <airport>)
     => (distance :: <single-float>) // Miles
      // Here, a squared plus b squared is equals to c squared, where c is the
      // hypotenuse, and a and b are the other sides of a right triangle
      sqrt((aircraft.altitude / $feet-per-mile) ^ 2
        + distance-2d(aircraft.current-position,
                      destination.current-position) ^ 2);
    end method distance-3d;

    // The distance between two positions, ignoring altitude
    define method distance-2d
        (position1 :: <relative-position>, position2 :: <absolute-position>)
     => (distance :: <single-float>) // Miles
      // When we have a relative position for the first argument (the
      // aircraft), we assume the relative position is relative to the second
      // argument (the airport)
      position1.distance;
    end method distance-2d;

    // It would be sensible to provide a distance-2d method that computed
    // the great-circle distance between two absolute positions
    // Our example does not need this computation, which is
    // beyond the scope of this book

    // The time it takes to go from the point of touchdown to the entrance
    // to the taxiway
    define method brake-time
        (aircraft :: <b707>, runway :: <runway>)
     => (duration :: <time-offset>)
      make(<time-offset>,
           total-seconds:
             ceiling/(runway.physical-size.length / $feet-per-mile,
                      $average-b707-brake-speed / $seconds-per-hour));
    end method brake-time;

    // The time it takes to go from the entrance of the taxiway to the point
    // of takeoff
    define method takeoff-time
         (aircraft :: <b707>, runway :: <runway>)
     => (duration :: <time-offset>)
      make(<time-offset>,
           total-seconds:
             ceiling/(runway.physical-size.length / $feet-per-mile,
                      $average-b707-takeoff-speed / $seconds-per-hour)
             + $takeoff-pause-time);
    end method takeoff-time;

    // The time it takes to taxi from the runway entrance across the taxiway
    // to the gate
    define method gate-time
        (aircraft :: <b707>, taxiway :: <taxiway>)
     => (duration :: <time-offset>)
      make(<time-offset>,
           total-seconds:
             ceiling/(taxiway.physical-size.length / $feet-per-mile,
                      $average-b707-taxi-speed / $seconds-per-hour));
    end method gate-time;

    // The time it takes to taxi from the gate across the taxiway to the
    // entrance of the runway
    define method runway-time
        (aircraft :: <b707>, taxiway :: <taxiway>)
     => (duration :: <time-offset>)
      gate-time(aircraft, taxiway);
    end method runway-time;

    // The time it takes to unload, service, and load an aircraft.
    define method gate-turnaround
        (aircraft :: <b707>, gate :: <gate>) => (duration :: <time-offset>)
      make(<time-offset>, total-seconds: $average-b707-gate-turnaround-time);
    end method gate-turnaround;

.. _nlanding-schedule-file:

The ``schedule.dylan`` file
---------------------------

This file contains the key generic functions and methods that compute
the schedule of aircraft transitions using the sorted sequence, time,
and position libraries, as well as the classes and methods described so
far in this chapter.

First, we present the five key generic functions that make up our
container protocol, followed by an implementation of that protocol for
the container classes defined in `Vehicle containers`_.

The container protocol and implementation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ``schedule.dylan`` file.

.. code-block:: dylan

    module: airport

    // The following generic functions constitute the essential protocol for
    // interaction between containers and vehicles

    // Returns true if container is available for aircraft in direction
    define generic available? (vehicle, container, direction);

    // Moves vehicle into container in the given direction
    define generic move-in-vehicle (vehicle, container, direction);

    // Moves vehicle out of container in the given direction
    define generic move-out-vehicle (vehicle, container, direction);

    // Returns the aircraft next in line to move out of container in direction
    define generic next-out (container, direction);

    // Returns the class of the next container to move vehicle into,
    // and how long it will take to get there
    define generic next-landing-step (container, vehicle);

    // A single storage container is available if the aircraft fits into the
    // the container, and there is not already a vehicle in the container
    define method available?
        (vehicle :: <aircraft>, container :: <single-storage>,
         direction :: <symbol>)
     => (container-available? :: <boolean>)
      object-fits?(vehicle, container)
        & ~ (container.vehicle-currently-occupying);
    end method available?;

    // A multiple storage container is available if the aircraft fits into
    // the container, and there are not too many aircraft already queued in
    // the container for the specified direction
    define method available?
        (vehicle :: <aircraft>, container :: <multiple-storage>,
         direction :: <symbol>)
     => (container-available? :: <boolean>)
      object-fits?(vehicle, container)
        & size(container.vehicles-by-direction[direction])
          < container.maxima-by-direction[direction];
    end method available?;

    // Avoids jamming the runway with inbound traffic, which would prevent
    // outbound aircraft from taking off
    // The runway is clear to inbound traffic only if there is space in the
    // next container inbound from the runway
    define method available?
        (vehicle :: <aircraft>, container :: <runway>,
         direction :: <symbol>)
     => (container-available? :: <boolean>)
      next-method()
        & select (direction)
            #"outbound" => #t;
            #"inbound"
              => let (class) = next-landing-step(container, vehicle);
                 if (class)
                   find-available-connection(container, class, vehicle) ~== #f;
                 end if;
          end select;
    end method available?;

    // A slot is used to keep track of which aircraft is in a single
    // storage container
    define method move-in-vehicle
        (vehicle :: <aircraft>, container :: <single-storage>,
         direction :: <symbol>)
     => ()
      container.vehicle-currently-occupying := vehicle;
      values();
    end method move-in-vehicle;

    // A deque is used to keep track of which aircraft are traveling in a
    // particular direction in a multiple storage container
    define method move-in-vehicle
        (vehicle :: <aircraft>, container :: <multiple-storage>,
         direction :: <symbol>)
     => ()
      let vehicles = container.vehicles-by-direction[direction];
      push-last(vehicles, vehicle);
      values();
    end method move-in-vehicle;

    // When an aircraft reaches the gate, it begins its outbound journey
    define method move-in-vehicle
        (vehicle :: <aircraft>, container :: <gate>,
         direction :: <symbol>)
     => ()
      next-method();
      vehicle.direction := #"outbound";
      values();
    end method move-in-vehicle;

    define method move-out-vehicle
        (vehicle :: <aircraft>, container :: <single-storage>,
         direction :: <symbol>)
     => ()
      container.vehicle-currently-occupying := #f;
      values();
    end method move-out-vehicle;

    define method move-out-vehicle
        (vehicle :: <aircraft>,
         container :: <multiple-storage>, direction :: <symbol>)
     => ()
      let vehicles = container.vehicles-by-direction[direction];
      // Assumes that aircraft always exit container in order, and
      // that this aircraft is next
      pop(vehicles);
      values();
    end method move-out-vehicle;

    // Determines what vehicle, if any, could move to the next container
    // If there is such a vehicle, then this method returns the vehicle,
    // the next container in the direction of travel,
    // and the time that it would take to make that transition
    define method next-out
        (container :: <vehicle-storage>, direction :: <symbol>)
     => (next-vehicle :: false-or(<vehicle>),
         next-storage :: false-or(<vehicle-storage>),
         time-to-execute :: false-or(<time-offset>));
      let next-vehicle = next-out-internal(container, direction);
      if (next-vehicle)
        let (class, time) = next-landing-step(container, next-vehicle);
        if (class)
          let next-container
            = find-available-connection(container, class, next-vehicle);
          if (next-container)
            values(next-vehicle, next-container, time);
          end if;
        end if;
      end if;
    end method next-out;

    // This method is just a helper method for the next-out method
    // We need different methods based on the class of container
    define method next-out-internal
        (container :: <single-storage>, desired-direction :: <symbol>)
     => (vehicle :: false-or(<aircraft>))
      let vehicle = container.vehicle-currently-occupying;
      if (vehicle & vehicle.direction == desired-direction) vehicle; end;
    end method next-out-internal;

    define method next-out-internal
        (container :: <multiple-storage>, desired-direction :: <symbol>)
     => (vehicle :: false-or(<aircraft>))
      let vehicle-queue = container.vehicles-by-direction[desired-direction];
      if (vehicle-queue.size > 0) vehicle-queue[0]; end;
    end method next-out-internal;

    // The following methods return the class of the next container to which a
    // vehicle can move from a particular container
    // They also return an estimate of how long that transition will take
    define method next-landing-step
        (storage :: <sky>, aircraft :: <aircraft>)
     => (next-class :: false-or(<class>), duration :: false-or(<time-offset>))
      if (aircraft.direction == #"inbound")
        values(<runway>, flying-time(aircraft, storage.airport-below));
      end if;
    end method next-landing-step;

    define method next-landing-step
        (storage :: <runway>, aircraft :: <aircraft>)
     => (next-class :: <class>, duration :: <time-offset>)
      select (aircraft.direction)
        #"inbound" => values(<taxiway>, brake-time(aircraft, storage));
        #"outbound" => values(<sky>, takeoff-time(aircraft, storage));
      end select;
    end method next-landing-step;

    define method next-landing-step
        (storage :: <taxiway>, aircraft :: <aircraft>)
     => (next-class :: <class>, duration :: <time-offset>)
      select (aircraft.direction)
        #"inbound" => values(<gate>, gate-time(aircraft, storage));
        #"outbound" => values(<runway>, runway-time(aircraft, storage));
      end select;
    end method next-landing-step;

    define method next-landing-step
        (storage :: <gate>, aircraft :: <aircraft>)
     => (next-class :: <class>, duration :: <time-offset>)
      values(<taxiway>, gate-turnaround(aircraft, storage));
    end method next-landing-step;

The scheduling algorithm
~~~~~~~~~~~~~~~~~~~~~~~~

The next methods form the core of the airport application.

The ``schedule.dylan`` file. *(continued)*

.. code-block:: dylan

    // Searches all of the vehicle storage of class class-of-next, which is
    // connected to container and has room for aircraft
    define method find-available-connection
        (storage :: <vehicle-storage>, class-of-next :: <class>,
         aircraft :: <aircraft>)
     => (next-container :: false-or(<vehicle-storage>))
      block (return)
        for (c in storage.connected-to)
          if (instance?(c, class-of-next)
              & available?(aircraft, c, aircraft.direction))
            return(c);
          end if;
        end for;
      end block;
    end method find-available-connection;

    // Generate new transitions to be considered for the next move
    // The transitions will be placed in the sorted sequence, which will order
    // them by earliest arrival time
    define method generate-new-transitions
        (container :: <vehicle-storage>,
         active-transitions :: <sorted-sequence>,
         containers-visited :: <object-table>)
     => ()
      unless(element(containers-visited, container, default: #f))
        // Keep track of which containers we have searched for new possible
        // transitions
        // We avoid looping forever by checking each container just once
        containers-visited[container] := #t;

        local method consider-transition (direction)
          // See whether any vehicle is ready to transition out of a container
          let (vehicle, next-container, time)
            = next-out(container, direction);
          unless (vehicle == #f | vehicle.next-transition.pending?)
            // If there is a vehicle ready, and it is not already in the
            // sorted sequence of pending transitions, then prepare the
            // transition instance associated with the vehicle
            let transition = vehicle.next-transition;
            transition.from-container := container;
            transition.to-container := next-container;

            // The vehicle may have been waiting
            // Take this situation into account when computing the earliest
            // arrival into the next container
            transition.earliest-arrival := transition.earliest-arrival + time;
            // Flag the vehicle as pending, to save searching through the
            // active-transitions sorted sequence later
            transition.pending? := #t;
            // Add the transition to the set to be considered
            add!(active-transitions, transition);
          end unless;
        end method consider-transition;

        // Consider both inbound and outbound traffic
        consider-transition(#"outbound");
        consider-transition(#"inbound");
        // Make sure that every container connected to this one is checked
        for (c in container.connected-to)
          generate-new-transitions(c, active-transitions, containers-visited);
        end for;
      end unless;
    end method generate-new-transitions;

    // Main loop of the program
    // See what possible transitions exist, then execute the earliest
    // transitions that can be completed
    // Returns the time of the last transition
    define method process-aircraft
        (airport :: <airport>, #key time = $midnight)
     => (time :: <time-of-day>)
      format-out("Detailed aircraft schedule for ");
      say(airport);
      format-out("\n\n");
      let sky = airport.sky-above;
      let containers-visited = make(<object-table>);
      let active-transitions = make(<sorted-sequence>,
      value-function: earliest-arrival);

      // We do not have to use return as the name of the exit procedure
      block (done)
        while (#t)
          // Each time through, start by considering every container
          fill!(containers-visited, #f);
          // For every container, see if any vehicles are ready to transition
          // If any are, add transition instances to the active-transitions
          // sorted sequence
          generate-new-transitions(sky, active-transitions,
                                   containers-visited);
    
          // If there are no more transitions, we have completed our task
          if (empty?(active-transitions)) done(); end;
          // Find the earliest transition that can complete, because there is
          // still room available in the destination container
          let transition-index
            = find-key(active-transitions,
                       method (transition)
                         available?(transition.transition-aircraft,
                                    transition.to-container,
                                    transition.transition-aircraft.direction);
                       end);
    
          // If none can complete, there is a problem with the simulation
          // This situation should never occur, but is useful for debugging
          // incorrect container configurations
          if (transition-index == #f)
            error("Pending transitions but none can complete.");
          end if;

          // Otherwise, the earliest transition that can complete has been
          // found: Execute the transition
          let transition = active-transitions[transition-index];
          let vehicle = transition.transition-aircraft;
          let vehicle-direction = vehicle.direction;
          move-out-vehicle(vehicle, transition.from-container,
                           vehicle-direction);
          move-in-vehicle(vehicle, transition.to-container, vehicle-direction);
    
          // This transition is complete; remove it from consideration
          transition.pending? := #f;
          remove!(active-transitions, transition);
          // Compute the actual time of arrival at the next container, and
          // display the message
          time := (transition.earliest-arrival := max(time, transition.earliest-arrival));
          say(transition);
          format-out("\n");
        end while;
      end block;
      time;
    end method process-aircraft;

The ``process-aircraft`` method uses components from the time, space and
sorted sequence libraries, the container classes and protocols, and the
vehicle classes and methods to schedule the aircraft arriving and
departing from an airport. The ``generate-new-transitions`` method assists
by examining the current state of all containers in the airport, and by
noting any new steps that vehicles could take.

.. _nlanding-airport-test-file:

The ``airport-test.dylan`` file
-------------------------------

The ``airport-test.dylan`` file contains test data, and the code that
constructs a model of the simple airport described in
:ref:`design-goals-airport-application`. The final method is a top-level
testing function that builds the airport model and executes the main
aircraft scheduling function. After defining the test, we show the results
of running it.

The ``airport-test.dylan`` file.

.. code-block:: dylan

    module: airport-test

    // To keep the example relatively simple, we will use variables to hold
    // test data for the flights and aircraft
    // Ordinarily, this information would be read from a file or database

    define variable *flight-numbers* = #[62, 7, 29, 12, 18, 44];

    define variable *aircraft-distances*
      = #[3.0, 10.0, 175.0, 450.0, 475.0, 477.0]; // Miles

    define variable *aircraft-headings*
      = #[82, 191, 49, 112, 27, 269]; // Degrees

    define variable *aircraft-altitudes*
      = #[7000, 15000, 22000, 22500, 22000, 21000]; // Feet

    define variable *aircraft-ids*
      = #["72914", "82290", "18317", "26630", "43651", "40819"];
 
    define constant $default-runway-size
      = make(<size>, length: 10000, width: 200, height: 100); // Feet

    define constant $default-taxiway-size
      = make(<size>, length: 900, width: 200, height: 100); // Feet

    // Assumes that there is only one runway, and one taxiway
    // The taxiway-count variable will determine how many aircraft can wait
    // in line for each direction of the taxiway
    define method build-simple-airport
        (#key gates-per-terminal :: <vector> = #[2],
              capacity :: <size> = $default-capacity,
              runway-size :: <size> = $default-runway-size,
              taxiway-size :: <size> = $default-taxiway-size,
              taxiway-count :: <positive-integer> = 5,
              position-report-time :: <time-of-day>
                = make(<time-of-day>,
              total-seconds: encode-total-seconds(6, 0, 0)))
     => (airport :: <airport>)
      let gates = generate-gates(gates-per-terminal, capacity);
      let taxiway
        = make(<taxiway>, id: "Echo", directions: #[#"inbound", #"outbound"],
               maxima: vector(taxiway-count, taxiway-count),
               capacity: capacity, physical-size: taxiway-size);
      let runway = make(<runway>, id: "11R-29L", capacity: capacity,
                        physical-size: runway-size);
      let keystone-air = make(<airline>, name: "Keystone Air", code: "KN");
      let flights
        = map(method (fn)
                make(<flight>, airline: keystone-air, number: fn) end,
              *flight-numbers*);

      let aircraft
        = map(method (aircraft-flight, aircraft-distance, aircraft-heading,
                      aircraft-altitude, aircraft-id)
                make(<b707>,
                     flight: aircraft-flight,
                     current-position:
                       make(<relative-position>,
                            distance: aircraft-distance,
                            angle:
                              make(<relative-angle>,
                                   total-seconds:
                                     encode-total-seconds
                                       (aircraft-heading, 0, 0))),
                     altitude: aircraft-altitude,
                     id: aircraft-id,
                     transition: make(<aircraft-transition>,
                                      arrival: position-report-time));
              end,
              flights, *aircraft-distances*, *aircraft-headings*,
              *aircraft-altitudes*, *aircraft-ids*);

      let airport
        = make(<airport>,
               name: "Belefonte Airport",
               code: "BLA",
               current-position:
                 make(<absolute-position>,
                      latitude:
                        make(<latitude>,
                             total-seconds: encode-total-seconds(40, 57, 43),
                             direction: #"north"),
                      longitude:
                        make(<longitude>,
                             total-seconds: encode-total-seconds(77, 40, 24),
                             direction: #"west")));

      let sky = make(<sky>, inbound-aircraft: aircraft, airport: airport,
                     id: concatenate("over ", airport.code));
      airport.sky-above := sky;
      runway.connected-to := vector(taxiway, sky);
      let taxiway-vector = vector(taxiway);
      for (gate in gates)
        gate.connected-to := taxiway-vector;
      end for;
      let runway-vector = vector(runway);
      taxiway.connected-to := concatenate(runway-vector, gates);
      sky.connected-to := runway-vector;
      airport;
    end method build-simple-airport;

    define method test-airport () => (last-transition :: <time-of-day>)
      process-aircraft(build-simple-airport());
    end method test-airport;

Now, we show the result of running ``test-airport``:

.. code-block:: dylan-console

    ? test-airport();
    => Detailed aircraft schedule for Airport BLA
    => 6:00: Flight KN 62 at Runway 11R-29L
    => 6:02: Flight KN 62 at Taxiway Echo
    => 6:02: Flight KN 7 at Runway 11R-29L
    => 6:03: Flight KN 62 at Gate A1
    => 6:04: Flight KN 7 at Taxiway Echo
    => 6:05: Flight KN 7 at Gate A2
    => 6:28: Flight KN 29 at Runway 11R-29L
    => 6:30: Flight KN 29 at Taxiway Echo
    => 6:37: Flight KN 62 at Taxiway Echo
    => 6:37: Flight KN 29 at Gate A1
    => 6:38: Flight KN 62 at Runway 11R-29L
    => 6:39: Flight KN 7 at Taxiway Echo
    => 6:42: Flight KN 62 at Sky over BLA
    => 6:42: Flight KN 7 at Runway 11R-29L
    => 6:46: Flight KN 7 at Sky over BLA
    => 7:11: Flight KN 29 at Taxiway Echo
    => 7:12: Flight KN 29 at Runway 11R-29L
    => 7:16: Flight KN 29 at Sky over BLA
    => 7:16: Flight KN 12 at Runway 11R-29L
    => 7:18: Flight KN 12 at Taxiway Echo
    => 7:18: Flight KN 18 at Runway 11R-29L
    => 7:19: Flight KN 12 at Gate A1
    => 7:20: Flight KN 18 at Taxiway Echo
    => 7:20: Flight KN 44 at Runway 11R-29L

    => 7:21: Flight KN 18 at Gate A2
    => 7:22: Flight KN 44 at Taxiway Echo
    => 7:53: Flight KN 12 at Taxiway Echo
    => 7:53: Flight KN 44 at Gate A1
    => 7:54: Flight KN 12 at Runway 11R-29L
    => 7:55: Flight KN 18 at Taxiway Echo
    => 7:58: Flight KN 12 at Sky over BLA
    => 7:58: Flight KN 18 at Runway 11R-29L
    => 8:02: Flight KN 18 at Sky over BLA
    => 8:27: Flight KN 44 at Taxiway Echo
    => 8:28: Flight KN 44 at Runway 11R-29L
    => 8:32: Flight KN 44 at Sky over BLA
    => {class <TIME-OF-DAY>}

The ``definitions-library.dylan`` file
--------------------------------------

The ``definitions-library.dylan`` file provides common definitions for all
the libraries in the airport example.

Note that this library and module, and the other libraries and modules
that follow, do not separate the library implementation module from the
library interface module, as discussed in :ref:`libraries-roles-of-modules`.
Dylan allows several different approaches to library and module architecture.
Here, we present an alternative organization.

The ``definitions-library.dylan`` file.

.. code-block:: dylan

    module: dylan-user

    define library definitions
      export definitions;
      use dylan;
    end library definitions;

    define module definitions
      export $letters, <positive-integer>;
      export $hours-per-day, $minutes-per-hour;
      export $seconds-per-minute, $seconds-per-hour, false-or;
      use dylan;
    end module definitions;

The ``definitions.lid`` file
----------------------------

The ``definitions.lid`` file.

.. code-block:: dylan

    library: definitions
    files: definitions-library
           definitions

The ``airport-library.dylan`` file
----------------------------------

The airport library implements the main scheduling system for the
airport example. This library assumes that your Dylan implementation
provides a ``format-out`` library, which supplies the ``format-out`` and
``format-to-string`` functions. This library also assumes that there is a
``transcendentals`` library, which supplies the ``sqrt`` (square root)
function.

The ``airport-library.dylan`` file.

.. code-block:: dylan

    module: dylan-user

    define library airport
      export airport;
      use dylan;

      use transcendentals;
      use say;
      use format-out;
      use definitions;
      use sorted-sequence;
      use angle;
      use time;
    end library airport;

    define module airport
      export <size>, length, height, width, current-position,
        current-position-setter;
      export physical-size, physical-size-setter, $default-capacity;
      export storage-capacity, storage-capacity-setter, identifier;
      export connected-to, connected-to-setter;
      export <gate>, generate-gates, <sky>, <runway>, <taxiway>;
      export <airline>, name, name-setter, code, code-setter, <flight>;
      export aircraft-flight, aircraft-flight-setter, number, number-setter,
        altitude, altitude-setter;
      export <aircraft-transition>, <b707>, <airport>, sky-above,
        sky-above-setter;
      export process-aircraft;
      use dylan;
      use transcendentals, import: {sqrt};
      use say;
      use format-out, import: {format-out};
      use format, import: {format-to-string};
      use definitions;
      use sorted-sequence;
      use time;
      use angle, export: {direction, direction-setter};
      use position;
    end module airport;

The ``airport.lid`` file
------------------------

The ``airport.lid`` file.

.. code-block:: dylan

    library: airport
    files: airport-library
           airport-classes
           vehicle-dynamics
           schedule

The ``airport-test-library.dylan`` file
---------------------------------------

The ``airport-test`` library implements a simple test case for the
scheduling system defined in the ``airport`` library.

The ``airport-test-library.dylan`` file.

.. code-block:: dylan

    module: dylan-user

    define library airport-test
      export airport-test;
      use dylan;
      use time;
      use angle;
      use airport;
    end library airport-test;

    define module airport-test
      export test-airport;
      use dylan;
      use time;
      use angle;
      use position;
      use airport;
    end module airport-test;

The ``airport-test.lid`` file
-----------------------------

The ``airport-test.lid`` file.

.. code-block:: dylan

    library: airport-test
    files: airport-test-library
           airport-test

Summary
-------

In this chapter, we presented a complete first draft of the airport
application, based on the techniques presented in previous chapters.
Although the example is complete and meets its stated design goals, we
can still make a number of improvements. For example, we could take
advantage of Dylan’s multiple inheritance to eliminate certain
repetitive slots. We could provide a container-implementor module
interface, and open the classes and generic functions so that users
could add their own classes of containers and extend the scope of the
application. We could take advantage of Dylan’s exception handling to
better deal with unusual situations that might occur during the
simulation. In the chapters that follow, we show the Dylan language
features that enable such improvements.
