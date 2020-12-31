<a name="top"></a>
## Groups (Angular 1.x)

This demonstration is a clone of the Groups demo, but using the Toolkit's Angular 1.x integration. 

![Groups Demo](demo-groups-app.png)

<a name="package"></a>
### package.json

```javascript
{
    "dependencies": {
        "font-awesome": "^4.7.0",
        "jsplumbtoolkit": "file:./jsplumbtoolkit.tgz",
        "jsplumbtoolkit-angular-1.x":"file:./jsplumbtoolkit-angular-1.x.tgz"
    }
}

```

[TOP](#top)

---

<a name="setup"></a>
### Page Setup

###### CSS

```xml
<link href="node_modules/font-awesome/css/font-awesome.min.css" rel="stylesheet">
<link rel="stylesheet" href="node_modules/jsplumbtoolkit/dist/css/jsplumbtoolkit-defaults.css">
<link rel="stylesheet" href="node_modules/jsplumbtoolkit/dist/css/jsplumbtoolkit-demo.css">
<link rel="stylesheet" href="app.css">
```
Font Awesome, `jsplumbtoolkit-demo.css`, and `app.css` are used for this demo and are not jsPlumb Toolkit requirements. `jsplumbtoolkit-defaults.css` is recommended for all apps using the Toolkit, at least when you first start to build your app. This stylesheet contains sane defaults for the various widgets in the Toolkit. 

###### JS

```xml
<script src="https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.4.5/angular.js"></script>
<script src="node_modules/jsplumbtoolkit/dist/js/jsplumbtoolkit.js"></script>
<script src="node_modules/jsplumbtoolkit-angular-1.x/dist/js/jsplumbtoolkit-angular-1.x.js"></script>
<script src="app.js"></script>
```

We import `jsplumbtoolkit.js` and `jsplumbtoolkit-angular` from `node_modules` (they are listed in `package.json`). `app.js` contains the demo code; it is discussed on this page.

[TOP](#top)

---

<a name="templates"></a>
### Templates

This is the template used to render Nodes: 

```xml
<script type="text/ng-template" id="node_template.tpl">
    <div>
        <div class="name">
            <div class="delete" title="Click to delete" ng-click="remove(node)">
                <i class="fa fa-times"></i>
            </div>
            <span>{% raw %}{{node.name}}{% endraw %}</span>
        </div>
        <div class="connect"></div>
    </div>
    <jtk-source filter=".connect"></jtk-source>
    <jtk-target></jtk-target>
</script>
```

This is the template used to render Groups:

```xml
<script type="text/ng-template" id="group_template.tpl">
    <div>
        <div class="group-title">
            {% raw %}{{group.title}}{% endraw %}
            <button class="expand" ng-click="toggleGroup(group)"></button>
        </div>
        <div jsplumb-group-content="true"></div>
    </div>
</script>
```

Group templates can have arbitrary markup as with Node templates. By default, the DOM element representing any Node that is a child of the Group will be appended to the Group's root element. You can, however, mark a place in the Group element that should act as the parent of Node elements - by setting `jsplumb-group-content="true"` on the element you wish to use. In this demo we use that concept to provide a title bar for each Group onto which Node elements can never
be dragged.

Note that there are a few main differences between the format of this template and that of a template using the Toolkit's default templating mechanism:

- You must prefix a `style` attribute, eg `ng-attr-style` instead of just `style`. This is just for IE compatibility; other browsers allow you to specify `style`.
- All references to the Node data that is being rendered are prefixed with `node.`. For Groups, Aal references to the Group data that is being rendered are prefixed with `group.`.

- Any `jtk-source`, `jtk-target` or `jtk-port` elements that are intended to configure a Node's main container have to be placed in the _root of the template_. This is in contrast to the default templating mechanism, which expects one root element per template. It is due to the fact that the template is rendered by Angular inside of its directive element, and the directive element is not discarded. There is a mechanism in Angular to instruct it to remove the original directive element (`replace:true` in a directive), but this is deprecated and not recommended by Angular, and will not work with the Toolkit integration.


[TOP](#top)

---

<a name="loading"></a>
### Data Loading

This is the data used by this demonstration:

```javascript
toolkit.load({
    data : {
        "groups":[
            {"id":"one", "title":"Group 1", "left":100, top:50 },
            {"id":"two", "title":"Group 2", "left":450, top:250, type:"constrained"  }
        ],
        "nodes": [
            { "id": "window1", "name": "1", "left": 10, "top": 20, group:"one" },
            { "id": "window2", "name": "2", "left": 140, "top": 50, group:"one" },
            { "id": "window3", "name": "3", "left": 450, "top": 50 },
            { "id": "window4", "name": "4", "left": 110, "top": 370 },
            { "id": "window5", "name": "5", "left": 140, "top": 150, group:"one" },
            { "id": "window6", "name": "6", "left": 50, "top": 50, group:"two" },
            { "id": "window7", "name": "7", "left": 50, "top": 450 }
        ],
        "edges": [
            { "source": "window1", "target": "window3" },
            { "source": "window1", "target": "window4" },
            { "source": "window3", "target": "window5" },
            { "source": "window5", "target": "window2" },
            { "source": "window4", "target": "window6" },
            { "source": "window6", "target": "window2" }
        ]
    }
});
```

As with Nodes, if you're using the `Absolute` layout, you can specify left/top properties for the element.

Additionally, Groups are considered to have a `type`, just like Nodes, whose default value is `default`, but which can be overridden in the same way as that of Nodes. Here we see the Group 2 is defined to be of type `constrained`, which we will discuss in the [View](#view) section below.

The relationship between Nodes and Groups is written into each Node's data, not into the Group data. Here we see that 4 of the Nodes in our dataset have a `group` declared.


[TOP](#top)

---

<a name="directives"></a>
### Directive Configuration

Everything is contained within one element that declares the controller and app to use:

```xml
<div class="jtk-demo-main" id="jtk-demo-groups" ng-controller="DemoController as DemoController" ng-app="app">
  ...
</div>
```

The demo uses three directives. Here we see how the Toolkit and its Miniview are declared:

```xml
<jsplumb-toolkit params="DemoController.toolkitParams"
             init="DemoController.init"
             render-params="DemoController.renderParams"
             jtk-id="demoToolkit"
             surface-id="demoSurface"
             style="width:750px;height:600px;position:relative;margin:0">

    ...

    <!-- miniview -->
    <jsplumb-miniview surface-id="demoSurface" class="miniview"></jsplumb-miniview>

</jsplumb-toolkit>
```

Points to note:

- Both `params` (for the Toolkit constructor) and `renderParams` (for the renderer) are provided.
- An ID is assigned to the Toolkit and to the Surface
- An `init` method is declared; this will be executed after the Toolkit and Surface have been created.

Here is how the palette (the set of nodes that can be dragged into the work area) is declared:

```xml
<div class="sidebar node-palette" jsplumb-palette selector="li" surface-id="demoSurface" type-extractor="DemoController.typeExtractor">
    <ul ng-repeat="obj in draggableTypes">
        <li jtk-node-type="{% raw %}{{ obj.type }}{% endraw %}" title="Drag to add new" jtk-group="{% raw %}{{ obj.group }}{% endraw %}">
            {% raw %}{{obj.label}}{% endraw %}
        </li>
    </ul>
</div>
```

Points to note:

- `draggableTypes` is an object in the scope of `DemoController`.
- `surface-id` maps to the `surface-id` specified in the `jsplumb-toolkit` directive.
- The `selector` attribute instructs the Palette to look for any `li` elements that are descendants of the Palette's element.
- A `type-extractor` function is specified; this is what determines the type of newly dropped elements. We set the `jtk-group` for the Group entry (it has a boolean `group:true` in `draggableTypes`).

#### Node/Group directives

As discussed in the [Angular integration](https://docs.jsplumbtoolkit.com/toolkit/current/articles/angular-1-x-integration) docs, each of your Node/Group types must be backed by a matching directive.  In this demo, we have a single Node type:

```javascript
app.directive('node', function (jsPlumbFactory) {
    return jsPlumbFactory.node({
        templateUrl: "node_template.tpl",
        inherit:["remove"]
    });
});
```

and a single Group type:

```javascript
app.directive('group', function (jsPlumbFactory) {
    return jsPlumbFactory.group({
        inherit:["toggleGroup"],
        templateUrl: "group_template.tpl"
    });
});
```

These directives map types to jsPlumb's Angular renderer, and take care of the vagaries of the digest loop for you, to ensure everything ends up painted properly. Note the `inherit` array on each of these: it is a list of method names to lift out of the controller and place into the scope of each directive when it is executed. This then allows you to reference the method in your HTML, for instance this from the Node template:

```xml
<div class="delete" title="Click to delete" ng-click="remove(node)">
    <i class="fa fa-times"></i>
</div>
```

We will discuss this again below in the [Behaviour](#behaviour) discussion.


[TOP](#top) 

---

<a name="model"></a>
### Model

The Toolkit instance is created with a `groupFactory` and a `nodeFactory`; these are the functions used when a new Group or Node is created after the user drags something on to the canvas:

```javascript
this.toolkitParams = {
    groupFactory:function(type, data, callback) {
        data.title = "Group " + (toolkit.getGroupCount() + 1);
        callback(data);
    },
    nodeFactory:function(type, data, callback) {
        data.name = (toolkit.getNodeCount() + 1);
        callback(data);
    }
};
```


[TOP](#top)

---

<a name="rendering"></a>
### Rendering

Render params are declared as members of `DemoController`, and referenced in the `jsplumb-toolkit` directive in the HTML:

```javascript
this.renderParams = {
    view : {
        nodes: {
            "default": {
                template: "node"
            }
        },
        groups:{
            "default":{
                template:"group",
                endpoint:"Blank",
                anchor:"Continuous",
                revert:false,
                orphan:true,
                constrain:false
            },
            constrained:{
                parent:"default",
                constrain:true
            }
        }
    },
    layout:{
        type:"Absolute"
    },
    jsPlumb: {
        Anchor:"Continuous",
        Endpoint: "Blank",
        //Endpoint: "Dot",
        Connector: [ "StateMachine", { cssClass: "connectorClass", hoverClass: "connectorHoverClass" } ],
        PaintStyle: { strokeWidth: 1, stroke: '#89bcde' },
        HoverPaintStyle: { stroke: "orange" },
        Overlays: [
            [ "Arrow", { fill: "#09098e", width: 10, length: 10, location: 1 } ]
        ]
    },
    lassoFilter: ".controls, .controls *, .miniview, .miniview *",
    dragOptions: {
        filter: ".delete *"
    },
    consumeRightClick:false
};
```

##### view

The single Node mapping is the most basic Node mapping possible; Nodes derive their Anchor and Endpoint definitions from the `jsPlumb` params passed in to the [render call](#rendering) discussed below.  

The Group mappings, though, bear a little discussion. First, the `default` Group mapping:

- *template* - ID of the template to use to render Groups of this type
- *endpoint* - Definition of the Endpoint to use for Connections to the Group when it is collapsed.
- *anchor* - Definition of the Anchor to use for Connections to the Group when it is collapsed.
- *revert* - Whether or not to revert Nodes back into the Group element if they were dropped outside.
- *orphan* - Whether or not to remove Nodes from the Group if they were dragged outside of it and dropped. In actual fact we did not need to set `revert` to false if `orphan` is set to true, but in this demo we included all the possible flags just for completeness.
- *constrain* - Whether or not to constrain Nodes from being dragged outside of the Group's element. 

The `constrained` Group mapping is declared to extend `default`, so it gets all of the values defined therein, but it overrides `constrain` to be true: Nodes cannot be dragged out of the Group element for this type of Group (in this demo we set Group 2 to be of type `constrained`, and Group 1 - and any Groups dragged on - to be of type `default`).

##### layout

Parameters for the layout. Here we specify an `Absolute` layout. It is the only layout currently that supports Groups.

##### lassoFilter

This selector specifies elements on which a mousedown should not cause the selection lasso to begin. In this demonstration we exclude the buttons in the top left and the miniview.

##### events

We listen for two events:

  `canvasClick` - a click somewhere on the widget's whitespace. Then we clear the Toolkit's current selection.
  
  `modeChanged` - Surface's mode has changed (either "select" or "pan"). We update the state of the buttons.


##### jsPlumb

Recall that the Surface widget is backed by an instance of jsPlumb Community Edition. This parameter sets the [Defaults](https://docs.jsplumbtoolkit.com/toolkit/current/articles/defaults.html) for that object.  


[TOP](#top)

---

<a name="initialisation"></a>
### Initialisation

`DemoController.init` looks like this:

```javascript
function(scope, element, attrs) {

    toolkit = scope.toolkit;

    toolkit.load({
        data: ...  (shown above) 
    });

    surface = jsPlumbService.getSurface("demoSurface");
    
    var controls = element[0].querySelector(".controls");
    // listener for mode change on renderer.
    surface.bind("modeChanged", function (mode) {
        jsPlumb.removeClass(controls.querySelectorAll("[mode]"), "selected-mode");
        jsPlumb.addClass(controls.querySelectorAll("[mode='" + mode + "']"), "selected-mode");
    });

    // pan mode/select mode
    jsPlumb.on(controls, "tap", "[mode]", function () {
        surface.setMode(this.getAttribute("mode"));
    });

    // on home button click, zoom content to fit.
    jsPlumb.on(controls, "tap", "[reset]", function () {
        toolkit.clearSelection();
        surface.zoomToFit();
    });
};

```

The main things to note here are that the `scope` passed in to this method contains `toolkit` and `surface` members. This demonstration assigns the `toolkit` member to a variable in the Controller's scope, so it is subsequently available everywhere.
 
[TOP](#top)

---

<a name="behaviour"></a>
### Behaviour

There are two pieces of behaviour that we need to code that are not completely handled for us by the Toolkit:  

- Delete Node
- Collapse/Expand Group

which is to say, the Toolkit's API provides functions to call to do these things, but there is no automatic binding of these functions to elements in the UI.

#### Delete Node

```xml
<div class="delete" title="Click to delete" ng-click="remove(node)">
    <i class="fa fa-times"></i>
</div>
```

```javascript
$scope.remove = function (obj) {
    toolkit.removeNode(obj);
};
```

`remove` makes a direct call to the `removeNode` method on the underlying Toolkit instance. The method is lifted into the Node directive's scope via the `node` directive that is discussed [above](#directives).


#### Collapse/Expand Group

```xml
<div class="group-title">
    {% raw %}{{group.title}}{% endraw %}
    <button class="expand" ng-click="toggleGroup(group)"></button>
</div>
```

```javascript
$scope.toggleGroup = function(group) {
    surface.toggleGroup(group);
};
```

As with `remove` above, `toggleGroup` is lifted into the directive's scope by dint of being declared in the `inherit` list for the `group` directive (see [Directive Configuration](#directives)).

[TOP](#top)

---

<a name="selecting"></a>
### Selecting Nodes

Lasso selection is enabled by default on the Surface widget. To activate the lasso, click the pencil icon in the toolbar:

![Lasso Select Mode](select-lasso.png)

The code that listens to clicks on this icon is as follows:

```javascript
// pan mode/select mode
jsPlumb.on(".controls", "tap", "[mode]", function () {
  renderer.setMode(this.getAttribute("mode"));
});
```


The tap listener extracts the desired mode from the button that was clicked and sets it on the renderer. This causes a `modeChanged` event to be fired, which is picked up by the `modeChanged` event listener in the View.

Note that here we could have used a `click` listener, but `tap` works better for mobile devices.

##### Lasso Operation

The lasso works in two ways: when you drag from left to right, any node that intersects your lasso will be selected.  When you drag from right to left, only nodes that are enclosed by your lasso will be selected.


##### Exiting Select Mode

The Surface widget automatically exits select mode once the user has selected something. In this application we also listen to clicks on the whitespace in the widget and switch back to pan mode when we detect one. This is the `events` argument to the `render` call:

```javascript
events: {
  canvasClick: function (e) {
    toolkit.clearSelection();
  }
}
```

`clearSelection` clears the current selection and switches back to Pan mode.

[TOP](#top)

---

<a name="adding"></a>
### Adding New Nodes/Groups

As discussed above, a `jsplumb-palette` is declared, which configures all of its child `li` elements to be droppable onto the Surface canvas.  When a drop occurs, the type of the newly dragged node is calculated by the `typeExtractor` function declared on `DemoController`:

```javascript
this.typeExtractor = function (el) {
    return el.getAttribute("jtk-node-type");
};
```

For a detailed discussion of this functionality, see [this page](https://docs.jsplumbtoolkit.com/toolkit/current/articles/rendering#dragpalette).

[TOP](#top)

---

<a name="deleting"></a>
### Deleting Nodes

![Single Node](demo-hierarchy-single-node.png)

Clicking the **X** button in this demonstration deletes the current node.

```javascript
jsPlumb.on("#canvas", "tap", ".delete *", function (e) {
  var info = toolkit.getObjectInfo(this);
  toolkit.removeNode(info.obj);
});
```

[TOP](#top)


---

<a name="collapsing"></a>
### Collapsing/Expanding Groups

Clicking the **-** button in this demonstration collapses a Group. It then changes to a +, which, when clicked, 
expands the Group.

```javascript
jsPlumb.on(canvasElement, "tap", ".group-title button", function(e) {
    var info = toolkit.getObjectInfo(this);
    if (info.obj) {
        renderer.toggleGroup(info.obj);
    }
});
```

The label of the button is changed via css: when a group is collapsed, it is assigned the CSS class `jsplumb-group-collapsed`. In the CSS for this demo we have these rules:
 
```css
.group-title button:after {
    content:"-";
}

.jtk-group.jtk-group-collapsed .group-title button:after {
    content:"+";
}
```

Another point to note is that the Toolkit does not take any specific action to "collapse" your Groups visually. It is left up to you to respond to the `jsplumb-group-collapsed` class as you need to. In this demo, we simply hide the group content area:

```css
.jtk-group.jtk-group-collapsed [jtk-group-content] {
    display:none;
}
```

[TOP](#top)
