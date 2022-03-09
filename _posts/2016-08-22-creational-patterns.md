---
layout: post
title: "Creational Patterns"
date: 2016-08-22 10:00:00
comments: true
keywords: "service architecture design"
categories:
- architecture
- design

tags:
- design pattern 
---

## ABSTRACT FACTORY

Provide an interface for creating **families** of related or dependent objects

### Scenario

- Client should behavior the same regardless of how its products are created, composed.
  - UI application doesn't care about how the button is rendered in Windows vs. OS X as long as they provides same interface
- Client can just change the ConcreteFactory to adopt the app to new behavior


### Participants

- AbstractFacory
  - declares an interface for operations that create abstract product objects
- ConcreteFactory
  - implements the operations to create concrete products
- AbstractProdcut
  - flares an interface for product
- ConcreteProduct
  - implement AbstractProdcut Interface
  - define the product created by concrete factory

![abscract_factory](/assets/images/abstract_factory.jpg)


### Example

```cpp
MazeGame game;
BombedMazeFactor factory;
game.CreateMaze(factory); // internall, use the facotry.MakeRoom/MakeWall, etc to create the game
```



## BUILDER

Separate the construction of a complex object from its representation so that the same construction process can create different representation.

### Scenario

- Construction process must allow different representations for the objects that's constructed
- The algorithm for creating a complex object should be independent of the parts that make up the object and how they're assembled
  - Maze game can be built with same steps but different type of rooms/wall blocks

### Participants

- Builder
  - define the abstract interface for creating parts of the product
- ConcreteBuilder
  - implement the Builder Interface
  - define and keeps track of the representation it creates
  - **provides interface for retrieving the products**
- Director
  - algorithm/steps to build the products with the builder

![builder](/assets/images/builder.jpg)

### Example

```cpp
Maze *maze;
MazeGame game; // director
StandardMazeBuilder builder; // builder

game.CreateMaze(builder);
maze = builder.GetMaze();
```



## FACTORY METHOD

Define an interface for creating an object, but let subclass decide which class to instantiate. It uses subclass/inheritance to solve the problem of creating different instances.

### Scenario

- class doesn't know the objects it must create
  - App needs to create documents but it doesn't know the type of the document
  - Some UserApp class needs to inherit base App class and overwrite the method on processing the documents

### Participants

- Product
  - define the interface of the object that the factory method creates
- ConcreteProduct
  - Implement the product interface
- Creator
  - define factory method which return an object of type Product.
- ConcreteCreator
  - overwrites the factory method to return the real instance of ConcreteProduct9kikk

![factory_method](/assets/images/factory_method.jpg)

### Example

```cpp
class MazeGame {
  public:
  	Maze* CreateMaze(); // game gets created here
  
  // factory methods:
  	virtual Room* MakeRoom(int n) ...
    virtual Wall* MakeWall...
}

class BombedMaze : public MazeGame {
  public:
  	virtual Wall* MakeWall() {
      return new BombedWall;
  	}
}
```



## PROTOTYPE

Give the client a prototypical instance. Client can use clone to create their own copy and start using the object.

### Scenarion

- Create an object has high cost
  - i.e., create a lot of IOs, complex algorithm, etc. Clone will be cheaper at this point
- A class only have a few different possible state. 
  - cheap to load it than creating from scratch
- classes needs to instantiate type of objects at run-time
  - dynamic loading

### Participants

- Prototype
  - clears an interface ==> Clone()
- ConcreatePrototype
  - implement clone

![prototype](/assets/images/prototype.jpg)

### Implementation

Can have a **prototype manager** to save all existing prototype and when client asks, just find it from the registry.

### Example

```cpp
class MazePrototypeFactory {
  public:
  	MazePrototypeFactory(Wall*, Room*, Door*);
  
  	Wall* MakeWall {
      return _prototypeWall->Clone();
  	}
}

MazeGame game;
MazePrototypeFactory MazePrototypeFactory(new BombedWall, new RoomWithABomb, new Door);
Maze* maze = game.CreateMaze(bombedMazeFactory)
```



## SINGLETON

Ensure a class only has one instance and provide a global point of access to it.

### Scenario

- there must be exactly one instance of a class and it should be shared between different clients
  - one window manager, one file system

### Implementation

- consider having a registry for singleton lookup
- [python singletonmixin](https://github.com/ning-yang/stock_tracer/blob/master/common/singletonmixin.py)
