# PrimeNGAdvancedGrowl

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [AdvGrowlModul](#advgrowlmodul)
  - [What is missing on PrimeNG?](#what-is-missing-on-primeng)
  - [What is the AdvGrowlModule offering?](#what-is-the-advgrowlmodule-offering)
  - [How do you use PrimeNGAdvancedGrowl?](#how-do-you-use-primengadvancedgrowl)
    - [installation](#installation)
    - [AdvGrowlComponent](#advgrowlcomponent)
      - [Input](#input)
      - [Output](#output)
    - [Models](#models)
    - [AdvGrowlService](#advgrowlservice)
    - [Examples](#examples)
      - [Avoid creating success messages with same Summary](#avoid-creating-success-messages-with-same-summary)
      - [Passing additional properties to the message](#passing-additional-properties-to-the-message)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# AdvGrowlModul
The AdvGrowlModule is a wrapper around the growl module from PrimeNG. This wrapper was created
because PrimeNG is missing some features.

## What is missing on PrimeNG?
- PrimeNG module does not offer a central service to
create growl messages. The PrimeNG message modul strongly couples the template and the component.
You need to include the growl component in each template.
- PrimeNG growl offers us to define a sticky property to remove the messages. When we set the lifetime
of the messages to 3 seconds all messages will be removed after the specified time. The problem comes
when a message gets created 2 seconds after the first message. This message will not be removed after
the specified 3 seconds. This message will be removed 1 seconds after creation. This is the point where
the 3 seconds from the first message have passed.

## What is the AdvGrowlModule offering?
- The AdvGrowlModule provides you the sticky feature with a unique lifetime for each message. The specified
lifetime is unique for each message. The growl message will only disappear after the given time has elapsed
or you pressed the cancel button on the growl message.
- The PrimeNGAdvancedGrowl module provides you a messageservice.
With the help of this service you have a central way to create growl messages.

## How do you use PrimeNGAdvancedGrowl?
### installation
PrimeNGAdvancedGrowl is an node_module and therefore it is provided over npm. To install it node and npm
are required.
```
npm install --save primeng-advanced-growl
```

To have all the primeNG styles available you need to import the following stylesheets in your application:
```
"../node_modules/font-awesome/scss/font-awesome.scss",
"../node_modules/primeng/resources/primeng.css",
"../node_modules/primeng/resources/themes/omega/theme.scss"
```

To use the AdvGrowlService and the AdvGrowlComponent you need to import the AdvGrowlModul in your appliction.
```javascript
import {AdvGrowlModule} from 'primeng-advanced-growl';

@NgModule({
    declarations: [AppComponent],
    imports: [AdvGrowlModule]
})
...
```


### AdvGrowlComponent
The AdvGrowlModule exports a component named AdvGrowlComponent. You need to include this component
once in your app.component.html. With the help of this component the advanced PrimeNG growl messages
can be displayed.
```html
<app-navbar></app-navbar>
<adv-growl></adv-growl>
<div class="container">
    <router-outlet></router-outlet>
</div>
```

The advanced growl messages component has the following in- and outputs.

#### Input
| Input        | Description                                                                                                                                                                                                                                                                                    |
|--------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| style        | Inline styles that should be applied to the growl component of PrimeNG                                                                                                                                                                                                                         |
| styleClass   | Style class for the growl component of PrimeNG                                                                                                                                                                                                                                                 |
| life: number | A number that represents the lifetime of each growl message. If set to 3000 each message will be disappear after 3 seconds. If no life param is passed to the components the growl messages are sticky and do not disappear until you call clearMessages or click on the cancel x on a message |

#### Output

| Event | Description |
|-------|-------------|
| onClose: EventEmitter<AdvPrimeMessage>| Emits an event with the closed message. This message is from type AdvPrimeMessage. |
| onClick: EventEmitter<AdvPrimeMessage>| Emits an event with the message from the clicked element. This message is from type AdvPrimeMessage. |
|onMessagesChanges: EventEmitter<Array<AdvPrimeMessage>>|Each time a message is created or removed it emits an array of all messages shown on the screen. If you subscribe to this emitter you always know which messages are on the screen.|

### Models
```javascript
export interface AdvPrimeMessage {
    id: string;
    severity: string;
    summary: string;
    detail: string;
    additionalProperties?: any;
}
```

### AdvGrowlService
The AdvGrowlService allows you to create and delete messages. The AdvGrowlService
can be accessed over dependency injection inside your component.

```javascript
import {AdvGrowlService} from 'primeng-advanced-growl';

@Component({
    selector: ...,
    templateUrl: ...
})
export class SampleComponent{

    constructor(private advGrowlService: AdvGrowlService){
    }
}
```

The AdvGrowlService provides the following methods to create messages. Each method expects
the message content and a message title.

- createSuccessMessage(messageContent: string, summary: string, additionalProperties?: any): void
- createInfoMessage(messageContent: string, summary: string, additionalProperties?: any): void
- createWarningMessage(messageContent: string, summary: string, additionalProperties?: any): void
- createErrorMessage(messageContent: string, summary: string, additionalProperties?: any): void

To clear all messages you can call the **clearMessages()** method from the AdvGrowlService.

### Examples
Inside the test folder you can find different examples how to use the library. Below we listed some code samples for common use cases.

#### Avoid creating success messages with same Summary
If we want to avoid the creation of multiple messages with the same summary or type we could do the following inside our component.
```javascript
import {AdvPrimeMessage} from '../../lib/messages/adv-growl.model';

@Component({
    selector: 'sample-app',
    template: `<adv-growl [life]="2500"
                (onMessagesChanges)="onMessages($event)">
               </adv-growl>
               <button pButton type="button"
                    (click)="createNonDuplicatedSuccessMessage()"
                    class="ui-button-success"
                    label="Create success message if none on screen">
               </button>
               `
})
export class AppComponent {

    messages = [];

    constructor(private advMessagesService: AdvGrowlService) {
    }

    public createSuccessMessage(): void {
        this.advMessagesService.createSuccessMessage('Awesome success message content', 'Awesome success', {
          clickMessage: 'Awesome click'
        });
    }

    public onMessages(messages) {
        this.messages = messages;
    }

    public createNonDuplicatedSuccessMessage(): void {
        const index = this.messages.findIndex(message => message.summary === 'Awesome success');
        if (index < 0) {
            this.createSuccessMessage()
        }
    }
}
```

#### Passing additional properties to the message
If you want to pass some additional informations to your message you can do this by passing those as last
optional parameter to the success, warning, info or error method. When you click on your message the event
will then contain your additionalProperties.
```javascript
import {AdvPrimeMessage} from '../../lib/messages/adv-growl.model';

@Component({
    selector: 'sample-app',
    template: `<adv-growl [life]="2500"
                (onMessagesChanges)="onMessages($event)"
                (onClick)="logMessage($event)">
               </adv-growl>
               <button pButton type="button"
                    (click)="createSuccessMessageWithAdditionalInfos()"
                    class="ui-button-success"
                    label="Create success message with additional properties">
               </button>
               `
})
export class AppComponent {

    messages = [];

    constructor(private advMessagesService: AdvGrowlService) {
    }

    public createSuccessMessageWithAdditionalInfos(): void {
        this.advMessagesService.createSuccessMessage('Awesome success message content', 'Awesome success', {
          clickMessage: 'Awesome click'
        });
    }

    public logMessage(message: AdvPrimeMessage) {
        if (message.additionalProperties) {
            console.log(message.additionalProperties.clickMessage)
        } else {
            console.log('You clicked on message', message)
        }
    }
}
```
