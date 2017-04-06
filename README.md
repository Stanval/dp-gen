# dp-gen

App for creation MVTS softswitch dialpeer configuration from tabular csv file.

Workflow:
1. Click "Choose file" button and select the file. (Sample data files could be found in the project root)
2. Every dialpeer section could have title prefix. The prefix could be applied via prefix input field.
3. Every dialpeer section could have list of allowed gateway groups. They could be specified via group input field.
4. Destination pattern are described with regular expressions. For readability purpose rows with dial patters could be limited in length. Default value is 60 characters.
5. Dialpeer could be created with diffirent priorities. Dialpeers with higher priorities overrides dialpeers with lower priorities in the dialpeer.cfg file.