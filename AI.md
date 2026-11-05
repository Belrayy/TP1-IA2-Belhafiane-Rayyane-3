# Utilisation de l'IA

Utilisation de claude afin de régler un problème d'affichage lors de l'activation du mode debug.

## Prompt

**Envoie du fichier, index.xhtml** et : **Can you fix this code so when the toggle is turned on it works**

## Réponse

The toggle does nothing because it never talks to the server. Two things are going wrong:

1. **No request is triggered.** `p:toggleSwitch` only changes the value in the browser. Without an AJAX behavior, `bb.debug` isn't updated until the next full form submit.
2. **`paneldebug` can't be updated directly.** It has `rendered="#{bb.debug}"`, and a component that isn't rendered doesn't exist in the HTML, so AJAX has nothing to replace. It needs an always-rendered wrapper.

## Fix

Wrap the debug panel in a container that is always rendered:

```xml
<!-- Always-rendered container so AJAX can update it -->
<h:panelGroup id="debugContainer" layout="block">
    <h:panelGrid id="paneldebug" columns="1" rendered="#{bb.debug}">
        <h:panelGroup>
            <h:outputText value="JSON envoyé dans la requête :"/>
            <button type="button" onclick="copyToClipboard('jsonrequete')">Copier requête JSON</button>
        </h:panelGroup>
        <h:inputTextarea id="jsonrequete" value="#{bb.texteRequeteJson}" title="Requête JSON"
                         cols="50" rows="15" readonly="true"/>
        <h:panelGroup>
            <h:outputText value="JSON retourné dans la réponse :"/>
            <button type="button" onclick="copyToClipboard('jsonreponse')">Copier réponse JSON</button>
        </h:panelGroup>
        <h:inputTextarea id="jsonreponse" value="#{bb.texteReponseJson}" title="Réponse JSON"
                         cols="50" rows="15" readonly="true"/>
    </h:panelGrid>
</h:panelGroup>
```

Add an AJAX behavior to the toggle:

```xml
<p:outputPanel style="display:flex; align-items: center; gap: 10px">
    <h:outputText value="Mode debug : "/>
    <p:toggleSwitch id="debugSwitch" value="#{bb.debug}" styleClass="mb-2" label="Mode debug">
        <p:ajax event="change" process="@this" update="debugContainer"/>
    </p:toggleSwitch>
    <p:tooltip for="debugSwitch"
               value="Active ou non l'affichage du JSON sur les requêtes et réponses du LLM." />
</p:outputPanel>
```

## Why `process="@this"`

Your `rolesysteme` menu is `required="true"`. If the AJAX request processed the whole form, toggling debug before choosing a role would fail validation and the toggle value would never reach the backing bean. `process="@this"` submits only the toggle.

## Check the backing bean

Make sure `bb` has a proper getter and setter for the property:

```java
private boolean debug;
public boolean isDebug() { return debug; }
public void setDebug(boolean debug) { this.debug = debug; }
```

If the bean is `@RequestScoped`, the conversation, `roleSysteme` and the other state will be lost between requests, including this AJAX call. It should be `@ViewScoped` (or `@SessionScoped`).
