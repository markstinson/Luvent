Luvent: A Simple Event Library for Lua
======================================

**Luvent is in early development and not recommended for any
  production software.  The API is subject to change and break
  compatibility without warning.**

Luvent is a library for [Lua][], written entirely in Lua, which helps
support [event-driven programming][EDP].  Luvent lets you create
events, which are objects with any number of associated functions.
Whenever you trigger an event the library will execute all functions
attached to that event.  You may trigger an event multiple times and
can provide different arguments that event’s functions each time.


Requirements
------------

Luvent requires one of the following Lua implementations:

* Lua 5.1 or 5.2
* [LuaJIT][] 2.0

These are the versions we use to test Luvent.  It should work with
later versions of each, and possibly older versions as well.

### Optional ###

The following programs are not necessary in order to use Luvent but
you will need them to run the unit tests, generate API documentation,
and so on.

* [LDoc][]
* [Busted][]


Installation
------------

All you need to do is place `src/Luvent.lua` in a directory that is part of
`package.path` so that Lua can find and load it.  If you have Busted
then you should run `make tests` first to ensure that Luvent behaves
as intended.


Documentation
-------------

Running the command `make docs` will populate the `docs/` directory
with HTML documents that describe Luvent’s API.  The public API
consists of the following functions and methods:

* `Luvent.newEvent()`
* `Luvent:addAction(action)`
* `Luvent:addActionWithInterval(action, time)`
* `Luvent:removeAction(action_or_id)`
* `Luvent:getActionCount()`
* `Luvent:callsAction(action)`
* `Luvent:trigger(...)`

**Note:** Developers must never rely on the properties of the return
  values of `Luvent:newEvent()`.  Its non-method properties are not
  part of the public API.

The parameter `action` can either be a function or a table that
implements the `__call()` metamethod.  Below is a lengthy example that
demonstrates the basics of creating and triggering events, and adding
and removing actions.

```lua
-- In this example we will pretend we are implementing a module in a
-- game that creates and manages enemies.  To simplfy the example we
-- use Enrique García Cota's terrific MiddleClass library in order
-- to make the class and objects for enemies.
--
--     https://github.com/kikito/middleclass
--
require "middleclass"

local Luvent = require "Luvent"
local Enemy = class("Enemy")

-- This hash contains a reference to all living enemies.
Enemy.static.LIVING = {}

function Enemy:initialize(family, maxHP)
    self.family = family
    self.maxHP = maxHP
    self.HP = maxHP
    table.insert(Enemy.LIVING, self)
end

-- This is the event we trigger any time an enemy dies.
Enemy.static.onDie = Luvent.newEvent()

-- This method applies damage to an enemy and will trigger its 'onDie'
-- event if the enemy's hit points reach zero or less.
function Enemy:damage(damage)
    self.HP = self.HP - damage
    if self.HP <= 0 then
        Enemy.onDie:trigger(self)
    end
end

-- Now we can start associating actions with the 'onDie' event.  First
-- we start by removing the enemy from the table of living enemies.
Enemy.onDie:addAction(
    function (enemy)
        for index,living_enemy in ipairs(Enemy.LIVING) do
            if enemy == living_enemy then
                table.remove(Enemy.LIVING, index)
                return
            end
        end
    end)

-- For debugging we want to see on the console when an enemy dies, so
-- we add that as a separate action.  This time we save the return
-- value of addAction() so that later we can use that to remove the
-- action when we want to stop printing debugging output.
local debugAction = Enemy.onDie:addAction(
    function (enemy)
        print(string.format("Enemy %s died", enemy.family))
    end)

-- Now we make some enemies and kill them to demonstrate how the
-- trigger() method used in Enemy:damage() invokes the actions.

local bee = Enemy:new("Bee", 10)
local ladybug = Enemy:new("Ladybug", 1)

-- This will print "2"
print(#Enemy.LIVING)

-- This kills the enemy so the program invokes the two actions above,
-- meaning it will print "Enemy Ladbug died" to the console and will
-- remove it from Enemy.LIVING.
ladybug:damage(100)
print(#Enemy.LIVING)

-- Now we turn off the debugging output by removing that action.  As a
-- result we will see no output after killing the bee.
Enemy.onDie:removeAction(debugAction)
bee:damage(50)
print(#Enemy.LIVING)
```


Acknowledgments
---------------

[EventLib][] by Elijah Frederickson is the major inspiration for the
design and implementation of Luvent.  The Luvent API also owes a debt
of ideas and names to [Node.js][] by Ryan Dahl et al.


Future Plans
------------

A future version of Luvent will support registering [coroutines][]
with events.  I may also implement support for collecting the return
values of functions registered with events.  However, neither of these
are critical features that I need from the first version of Luvent,
and so I want to take my time thinking about the API of implementation
of said features.


License
-------

[The MIT License](http://opensource.org/licenses/MIT)

Copyright 2013 Eric James Michael Ritz



[Lua]: http://lua.org/
[EDP]: http://en.wikipedia.org/wiki/Event-driven_programming
[EventLib]: https://github.com/mlnlover11/EventLib
[Node.js]: http://nodejs.org/
[LuaJIT]: http://luajit.org/
[LDoc]: http://stevedonovan.github.io/ldoc/
[Busted]: http://olivinelabs.com/busted/
[coroutines]: http://www.lua.org/manual/5.2/manual.html#2.6
