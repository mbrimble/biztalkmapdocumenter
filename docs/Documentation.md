## Instructions

# Put the [Microsoft command line transformation utility](http://www.microsoft.com/downloads/details.aspx?FamilyID=2fb55371-c94e-4373-b0e9-db4816552e41&DisplayLang=en) (msxsl.exe) file in the same folder as the BizTalkMapDocumenterHTML.xslt
# In the command prompt, navigate to the folder containing the msxsl.exe file
# Run and a command like the following:
* Either create an HTML file for a single btm map file:
{{
  msxsl "..\..\..\Source\Acme\Integration\Maps\SouceSystemA\MyMap.btm" BizTalkMapDocumenterHTML.xslt -o "MyMap.btm.html"
}}
* Or create an html file for each map file in a specified folder: 
{{
  for %%f in ("..\..\..\Source\Acme\Integration\Maps\SouceSystemA\*.btm") do call msxsl "%%f" BizTalkMapDocumenterHTML.xslt -o "%%~nf.btm.html"
}}
* Or you can add the following configuration to MSBuild to automate documentation of all maps as part of an build process:
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