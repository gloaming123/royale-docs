---
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
# 
# http://www.apache.org/licenses/LICENSE-2.0
# 
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

layout: docpage
title: Build the application
---

# Build the application

In many other HTML/JS/CSS development models, you write the JS and then just view it in the browser.  Royale uses a compiler to convert your MXML and ActionScript code into HTML/JS/CSS.  Why?  Because there is a philosophy that the sooner you catch a bug, the less expensive it is to fix it.  So the compiler scans all of your source code to make sure that it makes sense.  The compiler checks that there aren't typos in property names, that if you are expecting a String you'll probably get one, and more.  

The main MXML file should now look like this:

```XML
<js:Application xmlns:fx="http://ns.adobe.com/mxml/2009"
xmlns:local="*"
xmlns:js="library://ns.apache.org/royale/express" 
>
<fx:Script>
<![CDATA[
    public var commits:Array = [];
    public var repos:Array;
    public var projectName:String;

    private function setConfig():void
    {
        repos = configurator.json.repos;
        projectName = configurator.json.projectName;
    }

    private var currentIndex:int = 0;
    
    private function fetchCommits():void
    {
        commitsService.addEventListener("complete", gotCommits);
        commitsService.url = "https://api.github.com/repos/" + repos[currentIndex] + "/commits";
        commitsService.send();
    }

    private function gotCommits(event:Event):void
    {
        currentIndex++;
        var results:Array = commitsService.json as Array;
        var n:int = results.length;
        for (var i:int = 0; i < n; i++)
        {
            var obj:Object = results[i];
            var data:Object = {};
            data.author = obj.commit.author.name;
            data.date = obj.commit.author.date;
            data.message = obj.commit.message;
            commits.push(data);
        }
        if (currentIndex < repos.length)
        fetchCommits();
    }
]]>
</fx:Script>
<js:HTTPService id="configurator" url="project.json" complete="setConfig();fetchCommits()" />
<js:HTTPService id="commitsService" />

<js:initialView>
    <js:VView>
        <js:Label text="{projectName} Commits Log"/>
        <js:DataGrid id="dg" dataProvider="{commits}">
            <js:columns>
                <js:DataGridColumn label="Date" dataField="date" columnWidth="15"/>
                <js:DataGridColumn label="Author" dataField="author" columnWidth="15" />
                <js:DataGridColumn label="Message" dataField="message" columnWidth="70" />
            </js:columns>
        </js:DataGrid>
        <js:MultilineLabel text="{commits[dg.selectedIndex].message}" />
    </js:VView>
</js:initialView>
</js:Application>
```

Before we compile this code, there are a few things we want to add.  First off, there is no way to know if a  "var" has changed.  So, we want to tell the compiler to convert the var into a get/set property so we can detect changes.  Otherwise, you will get a warning that assignment to some variable cannot be detected.  In Royale, this is done via MetaData.  Metadata consists of square brackets "[]" which contain a tag and some attributes.  The compiler knows to act on some MetaData.  Other MetaData can be compiled into the code and used at runtime.  To convert a var to a get/set property use the [Bindable] metadata.  You can see it in the final code below.

Another thing missing is that we haven't specified the size of the controls.  Just like HTML/JS/CSS there are multiple ways to do this.  We could have a separate .css file and specify styles there and assign class names in the MXML tags.  That allows the CSS to be tweaked without recompiling so you can just refresh the browser and see changes, but in Royale, the CSS file is actually generated by the compiler from the CSS file in your source code and CSS files in the libraries you used so if you do make changes directly to the output CSS file, you will need to also make those changes in the source CSS file before compiling the source again.

This simple example will look fine if we just set the width and height of the DataGrid and the width of the MultilineLabel, so we will add them directly.  You can read the section on Styles to see other ways of setting styles on the View.

The final code should look like this:

```XML
<js:Application xmlns:fx="http://ns.adobe.com/mxml/2009"
xmlns:local="*"
xmlns:js="library://ns.apache.org/royale/express" 
>
<fx:Script>
<![CDATA[
[Bindable]
public var commits:Array = [];
public var repos:Array;
[Bindable]
public var projectName:String;

private function setConfig():void
{
repos = configurator.data.repos;
projectName = configurator.data.projectName;
}

private var currentIndex:int = 0;

private function fetchCommits():void
{
commitsService.addEventListener("complete", gotCommits);
commitsService.source = repos[currentIndex];
commitsService.send();
}

private function gotCommits(event:Event):void
{
currentIndex++;
commits = commits.concat(commitsService.data.commits);
if (currentIndex < repos.length)
fetchCommits();
}
]]>
</fx:Script>
<js:HTTPService id="configurator" source="project.json" complete="setConfig();fetchCommits()" />
<js:HTTPService id="commitsService" source="project.json" complete="setConfig();fetchCommits()" />

<js:initialView>
<js:VView>
<js:Label text="{projectName} Commits Log">
<js:DataGrid id="dg" dataProvider="commits" width="80%" height="300">
<js:columns>
<js:DataGridColumn label="Date" dataField="date" columnWidth="15"/>
<js:DataGridColumn label="Author" dataField="author" columnWidth="15" />
<js:DataGridColumn label="Message" dataField="message" columnWidth="70" />
</js:columns>
</js:DataGrid>
<js:MultilineLabel text="{dg.selectedItem.message}" width="80%"/>
</js:VView>
</js:initialView>
</js:Application>
```

Since we are intersted in JS output, to compile this code, we run:

```
    <path to Royale SDK>/royale-asjs/js/bin/mxmlc -debug=true GitHubCommitsViewer.mxml
```

If you've used NPM to install Royale, you can just run:

```
    mxmlc -debug=true GitHubCommitsViewer.mxml
```

This should compile without errors.  Next, let's see the results.

{:align="center"}
[Previous Page](create-an-application/application-tutorial/build.html) \| [Next Page](create-an-application/application-tutorial/debug.html)