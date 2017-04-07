Originally Posted by Steve Hart [http://www.hartsteve.com/2006/02/20/biztalk-map-documenter/](http://www.hartsteve.com/2006/02/20/biztalk-map-documenter/).

One of the frustrations with BizTalk maps is the obscurity of functoids when viewing maps in Visual Studio.  Depending on the volume of functoids in a map and/or depending on the degree of functoid chaining, easily understanding functoid usage can be impossible without diving into the functoid properties.  The BizTalk Map Documenter stylesheet will transform a a BizTalk map into an HTML view containing the following information:

* The node links between source and destination schemas.
* All functoids used in the map are displayed with their parameters.
* Chained functoids are displayed as nested “function” calls.
* Labels
* Constants
* For HL7 schemas where source and desitnation are the same message type, links between different nodes are highlighted. The way I do this is not reliable as it depends on detecting a pattern in the root node name; looks for node names ending in ‘GLO_DEF’. Is there a more reliable way?
The other advantage to having HTML views of your maps is that HTML is a much more friendly format to share with non-developers who need to understand the transformations (e.g. testers) and who may not have access to Visual Studio.

Here are some examples: 
* [ADTA01_Canonical_To_Lab.html](Home_ADTA01_Canonical_To_Lab.html) - version 1
* [MyCompanyJournalv1ToDynamicsAxGeneralJournalCreate.html](Home_MyCompanyJournalv1ToDynamicsAxGeneralJournalCreate.html) - version 2

The stylesheet can be run using the [Microsoft command line transformation utility (msxsl)](http://www.microsoft.com/downloads/details.aspx?FamilyID=2fb55371-c94e-4373-b0e9-db4816552e41&DisplayLang=en). Other more seamless approaches would be:

* A right-click explorer context menu for “.btm” files that would present a menu item to transform the map into an HTML view.
* Integration into Visual Studio so that you can choose to open a map as an HTML view (can’t remember how best to do this but I’m sure it can be done).
I’m sure there are a number of improvement/enhancements that can be made (e.g. including the functoid/link label attributes in the output). So please feel free to provide any feedback/ideas you have.

This can run as part of an MSBuild process to automatically create an HTML file for each of your BizTalk maps.  For example:

{code:xml}
<Target Name="CreateBizTalkMapDocumentation">
	<!-- Identify all of the map files - we'll include the project directories of all BizTalk projects, just to be sure -->
	<CreateItem Include="%(BizTalkProject.ProjectDir)\****\**.btm">
              <Output TaskParameter="Include" ItemName="BizTalkMapFiles"/>
        </CreateItem>
	<Message Text="BizTalk Server maps to document: @(BizTalkMapFiles)"/>
        <!-- Re-create the folder to hold the maps -->
	<RemoveDir Condition="Exists('$(MapDocumentationRoot)')" Directories="$(MapDocumentationRoot)" />
	<MakeDir Directories="$(MapDocumentationRoot)" />
	<!-- All the HTML files are written to a single folder, which assumes that each .btm file has a unique filename! -->
	<Exec Command='"$(MapDocumenterDir)\msxsl" "%(BizTalkMapFiles.FullPath)" "$(MapDocumenterDir)\BizTalkMapDocumenterHTML.xslt" -o "$(MapDocumentationRoot)\%(BizTalkMapFiles.Filename).html"' />
</Target>
{code:xml}