---
layout: post
title: "Structural Patterns - Part 2"
date: 2016-08-25 10:00:00
comments: true
keywords: "service architecture design"
categories:
- architecture
- design

tags:
- design pattern 
---

## DECORATOR

Add additional responsibilities to an object dynamically. The decorator can be reused and shared.

### Scenario

- add responsibilities to individual objects dynamically and transparently without affecting the other objects
- for responsibilities that can be withdrawn
- when extension by subclassing is impractical, i.e., will result in too many subclasses

### Participants

- Component
  - defines the interface for objects that can have responsibilities to be added
- ConcreteComponent
  - defines an object that additional responsibilities can be attached
- Decorator
  - maintains a reference to a Component object 
  - this can be **omitted** if only one decorator class is needed
- ConcreteDecorator
  - adds responsibilities to the component

![decorator](/assets/images/decorator.jpg)

### Example

```cpp
class VisualComponent {
  virtual void Draw();
}

class Decorator : public VisualComponent {
  virtual void Draw() {
    _component->Draw()
  }
  
  private:
  	VisualComponent* _component;
}

class BorderDecorator : public Decorator {
  virtual void Draw() {
    Decorator::Draw();
    DrawBorder()
  }
}

windows->SecContents(
	new BorderDecorator(
    	textView)
);
```



## FACEDE

Provide a unified interface to a set of interfaces in a subsystem.

### Scenario

- Provide interface to a complex subsystem. The interface could provide common function that most clients want. If client wants more, they can still use the subsystem interface directly
- decouple the subsystem from clients and other subsystem

### Participants

- Facade
  - knows which subsystem classes are responsible for a request
  - delegates client requests to appropriate subsystem objects
- subsystem classes
  - implement subsystem functionality

![facade](/assets/images/facade.jpg)

### Example

```cpp
class Scaner {...}
class Parser {...}
...
class compiler {
  // facade class
  virtual void Compile(istream& input, BytecodeStream& output) {
    Scanner scanner(input);
    Parser paser;
    paser.Parse(scanner, builder);
    ...
  }
}
```



## FLYWEIGHT

User sharing to support large numbers of fin-grained objects efficiently.

### Scenario

- An application users a large number of objects
  - i.e., text edit, needs object for each characters
- Most object state can be made extrinsic
  - character has intrinsic ==> c, which is store in flyweight
  - extrinsic ==> location, font, are stored in Tree structure
- Many groups of objects may be replaced by relatively few shared objects once extrinsic state is removed
- the application doesn't depend on object identity. 
  - since they doesn't have object but share the flyweight objects.

### Participants

- Flyweight
  - declares an interface through with flyweights can receive and act on extrinsic state
- ConcreteFlyweight
  - implements Flyweight interface and adds storage for **intrinsic** state if any.
  - i.e., store how to render one character
- FlyweightFactory
  - create an damages flyweight objects
  - may sure only one object is created and shared between clients

![flyweight](/assets/images/flyweight.jpg)

### Example

```cpp
class Charactor : public Glyph {
  virtual void Draw(Window*, GlyphContext&); // GlyphContext is the extrinsic state
  
  char _charcode; // intrinsic state
}
```



## PROXY

Provide a placeholder for another object to control access to it. 

### Scenaro

- **Remote Proxy** can provide local representation for an boject
- **Virtual Proxy** create expensive objects on demand. Lazy loading. 
  - i.e., create a placeholder for img, only load it when it is in the view
- **Protection Proxy** control access to the source object

### Participants

- Proxy
  - maintains a reference to the real subject
  - sometimes, overloading some method in real object
- Subject
  - define common interface for real subject and proxy so proxy can be used everywhere the real object is used
- RealSubject
  - define the real object that proxy represent

![proxy](/assets/images/proxy.jpg)

### Example

```cpp
class Graphic {...}
class ImageProxy : public Graphic {
  
  void Draw() {
    GetImage()->Draw();
  }
  
  Image* GetImage() {
    if (_image == 0) {
      _image = new Image(_fileName);
    }
    return _image;
  }
private:
  char* _filename;
  Image* _images;
}
```



