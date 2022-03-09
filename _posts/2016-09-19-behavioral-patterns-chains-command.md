---
layout: post
title: "Behavioral Patterns - Chain of Responsibility, Command"
date: 2016-09-19 10:00:00
comments: true
keywords: "service architecture design"
categories:
- architecture
- design

tags:
- design pattern 
---

## CHAIN OF PESPONSIBILITY

When a request comes, chain the receiving objects and pass the request along the chain until an object handles it.

### Scenario

- more than one object may handle a request
- set of objects that can handle the rest should be specified dynamically

### Participants

- Handler

  - defines an interface for handling requests
  - (optional) implements the successor link, i.e.,

  ```c++
  class HelpHandler {
  public:
    virtual void HandleHelp();
  private:
    HelpHanlder* _successor;
  }

  void HelpHandler::HandleHelp() {
    if (_successor) {
      _successor->HandleHelp()
    }
  }
  ```

  if the subclass is not interested in handling the request, it can just leave the handle method default and it will gets passed to next successor.

- ConcreteHandler

  - if it can handle the request, it does so; otherwise, it forwards the request to its successor

- Clinet

  - initiates the request to ConcreteHandler object 

![chain_of_responsibility](/assets/images/chain_of_responsibility.jpg)

### Example

```cpp
Application* application = new Application();
Dialog* dialog = new Dialog(application);
Button* button = new Button(dialog);

button->HandleHelp();
```



## COMMAND

Bundle command, args, receiver into an object which can be executed later. It allows generic handling of different request, queuing those request and support undoable operations.

## Scenario

- specify, queue and execute requests at different times
- support undo
- logging changes so they can be reapplied in case of system crash
- for transaction system

### Participants

- Command
  - declares an interface for execution an operation - Execute, Undo, etc
- ConcreteCommand
  - defines a binding between a Receiver object and an action
  - implements Execute by invoking the corresponding operations on Receiver
- Client
  - create ConcreteCommand object and sets its receiver
- Invoker
  - ask command to execute the request

![command](/assets/images/command1.jpg)
![command](/assets/images/command2.jpg)

### Example

```cpp
class Command {
  ...
  virtual void Execute() = 0;
}

class OpenFileCommand: public Command {
 	OpenFileCommand(Application*);
  	virtual void Execute();
}

void OpenFileCommand::Execute() {
  char* name = AskUser();
  
  if (name != 0) {
    ....
    document->Open();
  }
}
```

