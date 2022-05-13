---
id: szp6r8w3ldllgnitjlwqi36
title: 'Code Review: Cooleg'
desc: ''
updated: 1652432487294
created: 1652414487503
---

## Information
#### Requestor
* Cooleg (308702273154973717)

#### Project Link
* https://github.com/awesmprogram/Civilization

#### Project Model
* OOP

## Code Review
I'll be experimenting around with the code-review layout, feel free to let me know if you got some feedback.

### Encapsulation:  
One of the fundamentals of OOP (Object-Oriented Programming) concepts is called Encapsulation. Encapsulation (or also referred to as Information Hiding) is in simple terms, a concept of locking behind your data, and have methods interacting with the data instead, that way you would retain the ability to keep track of what's happening.

By not applying that concept to your code design, you're essentially giving up control of your data to the hands of other classes, and that's not good.

If you're wondering why that's the case, let me create a scenario to explain the cons.

#### Scenario: Wanting to print out the data
Let's say we wanted to print out the data before and after every time it's updated for debugging purposes.

##### Not Following Encapsulation Pattern
```java
    public class Person {
        public int age;

        public Person(final int age) {
            this.age = age;
        }

        // not possible since we gave up control of our data
    }

    public class RandomClass {
        public modifyPersonAge() {
            final Person person = new Person(5);
            person.age = 6;
        }
    }
```

##### Following Encapsulation Pattern
```java
    public class Person {
        // retained control of our data
        private int age;

        public Person(final int age) {
            this.age = age;
        }

        // added a way to update the age, but with our own way
        public void setAge(final int age) {
            Logger.debug("Before " + age);
            this.age = age;
            Logger.debug("After " + age);
        }
    }

    public class RandomClass {
        public modifyPersonAge() {
            final Person person = new Person(5);
            person.setAge(6); // > before: 5 > After: 6
        }
    }
```

It's a pretty simple concept in hindsight, but it's a fundamental one to say the least.

### References, Singleton and Static:
I believe that the `Manage` reference class should either, be cached somewhere, transformed into a Singleton, or turn the `ItemStack`'s to be constant.

As far as I can tell from the code, the ItemStack that was created was not bound to change in runtime (when the code is running), so it should be a constant. We'll be utilizing something called `Static Initialization Block`.

Example can be seen in https://gist.github.com/Tofpu/343bd4c79bde5129f38dfe987e122e9d or here:
```java
  final ItemStack PVP_TOGGLE_ENABLED;
  final ItemStack PVP_TOGGLE_DISABLED;

  static {
    PVP_TOGGLE_ENABLED = new ItemStack(Material.GREEN_CONCRETE);
    final ItemMeta pvpToggledEnabledMeta = pvptoggle.getItemMeta();
    pvpToggledEnabledMeta.setDisplayName(ChatColor.GREEN + "PVP is currently enabled!");

    pvpToggledEnabledMeta.setLore(Collections.singletonList("Click to disable PVP");
    PVP_TOGGLE_ENABLED.setItemMeta(pvpToggledEnabledMeta);

    PVP_TOGGLE_DISABLED = new ItemStack(Material.GREEN_CONCRETE);
    final ItemMeta pvpToggledDisabledMeta = pvptoggle.getItemMeta();
    pvpToggledDisabledMeta.setDisplayName(ChatColor.GREEN + "PVP is currently disabled!");

    pvpToggledDisabledMeta.setLore(Collections.singletonList("Click to enable PVP");
    PVP_TOGGLE_DISABLED.setItemMeta(pvpToggledDisabledMeta);
```

#### Reasoning
My personal belief is that if a new instance of a class has been called twice at any given moment, the new instance is (or should be) uniquely different to the previous instance that was created - and if it weren't, it should be cached, or transformed into a Singleton.

I might've missed to take something into account. (let me know if I did)

### Running I/O Operations on Main Thread:
Running I/O operations on the main thread is heavily forbidden upon, as it'll force the main thread to wait until the operation is complete - and by nature, I/O operations are very resource intensive.

I see that the config is being saved in the main thread, and as I mentioned above, it's a really bad practice. Look to the references below to learn how to save the config (or anything that IO/Network/thread-safe) async.

## Readability
### Class Naming:
I noticed that some of the classes naming are quite misleading:
- CivUtils -> CivPlugin
- BorderUtils -> BorderHandler
- ...etc

The class names should aline with the class responsibilities for readability reasons. If the class is responsible for registering items, it should mention Registry somewhere in the class name (or something similar), and etc.

### Long methods: 
I see that the `ChatColor.translateAlternateColorCodes` method is being utilized directly, I'd highly recommend you to create a util class that would call the mentioned method for readability purposes.

### Naming Convention: 
It seems like you forgot to follow the Java Naming Convention on Manage class.

## Other
### Multiple Lookups: 
I see that there's unnecessary lookup operations occurring, which could simply be cut down to a single lookup operation by caching said result to a variable. (referring `BorderUtils` class).

It might seem to be insignificant at the moment, but that won't always be the case when you're dealing with concurrent operations (which will cost you a lot of time debugging the issue)

Example can be seen in <https://gist.github.com/Tofpu/2af109422b32b02ed33b421281751910> or here:
```java
  final Map<String, Integer> scoreMap = new HashMap<>();
  scoreMap.put("Tofpu", 69);
  
  // doing multiple lookup to scoreMap, bad!
  if (scoreMap.get("Tofpu") > 69) {
      final Integer score = scoreMap.get("Tofpu");
      scoreMap.put("Tofpu", score + 1);
  }

  // decreased the lookup operation to one, good!
  final Integer score = scoreMap.get("Tofpu");
  if (score > 69) {
      scoreMap.put("Tofpu", score + 1);
  }
```

### == & .equals: 
Comparing an Enum with `.equals` is a really bad practice, and is not correct whatsoever. It should be changed to `==` - as a matter of fact, Enums are integers behind the scenes, and integers are supposed to be compared with `==`, not with `.equals`.

## References:
- Encapsulation - <https://refactoring.guru/encapsulate-field>
- Bukkit Scheduler - <https://bukkit.fandom.com/wiki/Scheduler_Programming#Example_2>
- `==` vs `.equals`- <https://stackoverflow.com/questions/7520432/what-is-the-difference-between-and-equals-in-java>
- Singleton - <https://refactoring.guru/design-patterns/singleton>
- Static Initialization Block - <https://stackoverflow.com/questions/335311/what-is-the-difference-between-a-static-and-a-non-static-initialization-code-blo>