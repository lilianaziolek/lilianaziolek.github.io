---
title:  "Solving Lighthouse 'Avoid an excessive DOM size' issue"
categories:
  - front-end
tags: 
  - vuejs
  - nuxt
  - performance
  - java
---
Recently we started looking a bit into [OSBO](https://www.onestopbeauty.online) performance. As the page was built mostly at the time when we didn't understand front-end development that well (British for: *we had no idea what we were doing*), plus we didn't have any active monitoring on performance, various problems obviously managed to sneak in.

# If you don't know Lighthouse, check it out first
There are plenty of articles on how to launch Lighthouse and they contain a number of very helpful suggestions, so I won't reiterate this here. There was one issue where the advice was not particularly friendly though: "Avoid an excessive DOM size". In our case, even our home and signup pages had around 3500 DOM nodes and, considering they are fairly simple, this sounded excessive. We struggled to understand where all these nodes were coming from. All the advice was around "avoid creating too many DOM nodes" - but I just couldn't find any useful info on how do I find out where (logically in my codebase) the nodes are created. Which part of my code is the problem? It's hard to optimise until you know which component(s) you need to optimise.

So, I quickly knocked out a tool to help us find the "DOM bottlenecks". And as I still lurrrrve Java (or rather: that's a tool I'm most productive in), it's in Java - sorry folks ;)

# Find the DOM branches to trim
The principle is actually really simple, and similar to how you'd go around finding where all the space on your hard drive goes if you suddenly run out of space. You find the biggest folder. Then the biggest folder in the biggest folder. And so on, until you see something suspicious - a folder bigger than you would normally expect.

In order to do that without spending too much time writing the tool itself (ultimately it took me maybe 30mins) I decided to use JSoup (to parse the DOM tree from our website), and Jackson - to print the results nicely, as I can then easily collapse/expand JSON in IntelliJ (helpful hint: open any .json file and press **CTRL-ALT-L** to nicely indent a single massive line of JSON).

The full result is in [Github Repo](https://github.com/lilianaziolek/blog-examples/tree/master/lighhouse-performance-issues) in folder **Lighthouse Performance Issues** (I have a feeling we might need more stuff to help us with Lighthouse reports).
There are two classes in the project:

<details><summary><b>OsboDomNode</b> - a class representing DOM in terms of what we care about: total number of child (and grand... child nodes), and some basic stats on direct children. It uses recursive functions to aggregate total number of nodes in each of the DOM elements.
</summary>
<p>

```java
package online.onestopbeauty.blog.examples.lighthouse.dom;

import com.fasterxml.jackson.annotation.JsonIgnore;
import com.fasterxml.jackson.annotation.JsonPropertyOrder;
import lombok.*;
import org.jsoup.nodes.Element;
import org.jsoup.select.Elements;

import java.util.List;
import java.util.Map;
import java.util.Optional;
import java.util.stream.Collectors;

import static java.util.Collections.emptyList;
import static java.util.Comparator.naturalOrder;
import static java.util.stream.Collectors.groupingBy;

@Data
@Builder
@JsonPropertyOrder({ "description", "type", "allChildNodesCount", "childNodesSummary" })
public class OsboDomNode {

    private final String type;
    private final String description;
    @JsonIgnore
    @Singular
    private final List<OsboDomNode> childNodes;

    @Getter(AccessLevel.NONE)
    private Integer allChildNodesCount;

    public int getAllChildNodesCount() {
        if (allChildNodesCount == null) {
            allChildNodesCount = this.childNodes.size() + this.childNodes.stream().mapToInt(OsboDomNode::getAllChildNodesCount).sum();
        }
        return allChildNodesCount;
    }

    public List<String> getChildNodesSummary() {
        Integer allChildNodesCount = this.getAllChildNodesCount();
        return this.childNodes.stream().map(child -> percentageInChild(child, allChildNodesCount)).collect(Collectors.toList());
    }

    public List<OsboDomNode> getNodesWithHighestNumberOfChildren() {
        Map<Integer, List<OsboDomNode>> nodesWithChildCount = childNodes.stream().collect(groupingBy(OsboDomNode::getAllChildNodesCount));
        Optional<Integer> maxNodes = nodesWithChildCount.keySet().stream().max(naturalOrder());
        if (maxNodes.isPresent()) {
            return nodesWithChildCount.get(maxNodes.get());
        } else {
            return emptyList();
        }
    }

    private String percentageInChild(OsboDomNode child, Integer allChildNodesCount) {
        double percentage = 100.0 * child.getAllChildNodesCount() / allChildNodesCount;
        return String.format("%d [%.2f%%] in %s %s", child.getAllChildNodesCount(), percentage, child.type, child.description);
    }

    public static OsboDomNode fromElement(Element element) {
        OsboDomNode.OsboDomNodeBuilder builder = OsboDomNode.builder();
        builder.type(element.tag().getName() + "[" + element.siblingIndex() + "]");
        builder.description(element.attributes().toString());

        Elements children = element.children();
        children.forEach(child -> builder.childNode(OsboDomNode.fromElement(child)));
        return builder.build();
    }
}
```

</p></details>

<details><summary><b>OsboPerfHelper</b> - a simple runner, you put in the URL of your website (could even be localhost), it goes off, reads the DOM structure, and then we feed it into the OsboDomNode to be analysed.</summary>
<p>

```java
package online.onestopbeauty.blog.examples.lighthouse.dom;

import com.fasterxml.jackson.databind.ObjectMapper;
import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.nodes.Element;

import java.io.File;
import java.io.IOException;

public class OsboPerfHelper {

    private static final ObjectMapper OBJECT_MAPPER = new ObjectMapper();

    public static void main(String[] args) throws IOException {
        String osboUrl = "http://localhost:8081";
        Document doc = Jsoup.connect(osboUrl).get();
        Element body = doc.body();
        OsboDomNode osboDomNode = OsboDomNode.fromElement(body);
        System.out.println((Integer) osboDomNode.getAllChildNodesCount());
        printJson(osboDomNode);

//        printJson(osboDomNode.getNodesWithHighestNumberOfChildren()
//                .get(0)
//                .getNodesWithHighestNumberOfChildren()
//                .get(0)
//                .getNodesWithHighestNumberOfChildren()
//                .get(0)
//                .getNodesWithHighestNumberOfChildren()
//                .get(0)
//                .getChildNodes()
//                .get(0));
    }

    private static void printJson(OsboDomNode osboDomNode) throws IOException {
//        System.out.println(OBJECT_MAPPER.writeValueAsString(osboDomNode));
        File resultFile = new File("domNode.json");
        OBJECT_MAPPER.writeValue(resultFile, osboDomNode);
        System.out.println("Written JSON result into " + resultFile.getAbsolutePath());
    }

}

```

</p></details>

<details><summary>Respective build.gradle file</summary>
<p>

```groovy
plugins {
    id 'java'
}

group 'online.onestopbeauty.blog.examples'
version '1.0-SNAPSHOT'

sourceCompatibility = 1.8

repositories {
    mavenCentral()
}

dependencies {
// https://mvnrepository.com/artifact/org.jsoup/jsoup
    compile group: 'org.jsoup', name: 'jsoup', version: '1.12.1'
// https://mvnrepository.com/artifact/org.projectlombok/lombok
    compileOnly group: 'org.projectlombok', name: 'lombok', version: '1.18.8'
// https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind
    compile group: 'com.fasterxml.jackson.core', name: 'jackson-databind', version: '2.9.9'


    testCompile group: 'junit', name: 'junit', version: '4.12'
}
```

</p></details>

Oh yes, I use Lombok for constructors, builders and other boilerplate (getters etc.) - just because Lombok is awesome and it's the first thing I always add to any Java project. Just remember to add Lombok plugin and turn on annotation processing in IntelliJ, otherwise you'll get compilation errors.

# Our real-world experience
So how did things look like for us when running on the live version?
The first few levels of nodes looked fairly healthy, with body and direct subnodes containing around 99% of nodes each (simply a few layers of wrappers, nothing to worry about).
But then I saw something suspicious (and here hat-tip to Vuetify for using meaningful class names in components - makes troubleshooting so much easier):
```json
{
"description": " class=\"application--wrap\"",
"type": "div[0]",
"allChildNodesCount": 3401,
"childNodesSummary": [
  "[39.05] in div[2]  class=\"layout\" data-v-3a808de6",
  "[56.40] in main[4]  class=\"v-content\" style=\"padding-top:0px;padding-right:0px;padding-bottom:56px;padding-left:0px;\"",
  "[4.38] in footer[6]  data-cy=\"osboFooter\" class=\"v-footer v-footer--absolute v-footer--inset theme--light\" style=\"height:auto;margin-bottom:56px;border-radius:10px;\" data-v-3645c51c",
  "[0.06] in button[8]  type=\"button\" medium=\"\" class=\"v-btn v-btn--bottom v-btn--floating v-btn--fixed v-btn--right v-btn--small theme--dark secondary fab-style\" style=\"display:none;\" data-v-045da490"
]}
```

The "main" part of our app was taking under 60% of the nodes, and the "div[2] / layout" element was taking nearly 40%.
At this point I added an extra log statement in the OsboPerfHelper, drilling down into the correct node. This of course could be done in a much nicer way, and if I have to use it more often perhaps I'd add some nicer "drill down" tooling - but at this point, it was a "quick and dirty" job of half an hour or so - and did the job well enough:
```java
printJson(osboDomNode.getNodesWithHighestNumberOfChildren().get(0)
                .getNodesWithHighestNumberOfChildren().get(0)
                .getNodesWithHighestNumberOfChildren().get(0)
                .getNodesWithHighestNumberOfChildren().get(0)
                .getChildNodes().get(0));
```

The result was:
```json
{
"description": " class=\"flex offset-md1 md10 xs12\" data-v-3a808de6",
"type": "div[0]",
"allChildNodesCount": 1327,
"childNodesSummary": [
   "[0.45] in div[0]  class=\"layout\" data-v-0c4978b8 data-v-3a808de6",
   "[65.49] in aside[2]  data-cy=\"mobileNavBar\" class=\"offWhite1 v-navigation-drawer v-navigation-drawer--clipped v-navigation-drawer--close v-navigation-drawer--fixed v-navigation-drawer--temporary theme--light\" style=\"height:100%;margin-top:0px;transform:translateX(-375px);width:375px;\" data-v-c332d172 data-v-3a808de6",
    "[33.84] in nav[4]  id=\"attachMenu\" data-cy=\"osboToolBar\" class=\" text-xs-center px-0 toolbarStyle v-toolbar elevation-0 v-toolbar--dense v-toolbar--extended theme--light\" style=\"margin-top:0px;padding-right:0px;padding-left:0px;transform:translateY(0px);\" data-v-3a808de6"
]}
``` 

This allowed me to see that I have nearly 900 nodes in my mobile navbar. The funny thing was that I don't even need mobileNavbar (with mobile version of the menu) on the desktop version of the page, which I was testing at this point. Thus, we went off and did some simple cleanups, to reduce the size of the mobile menu (900 nodes sounds excessive even when it's needed), and to make sure it's not generated on desktops (as it's a waste and never displayed).

This was just the beginning of trimming down the DOM tree (we're now on around 1700 nodes on localhost, so massive reduction, and still more to come) - but the key was to know which DOM "branches" to trim.

# Is there anything better out there?
If you know a better tool for this job, please leave a note in the comments below. I find it very hard to believe that such a simple problem doesn't have something already existing - but a quick google search gave me mostly results with many articles describing why large DOM is bad - not how to find your worst offenders in the DOM tree. Otherwise, feel free to report if this "micro-tool" was helpful in any way.
