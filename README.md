//Using Webhook and Pipedream

//8 code steps are created.

//1.Trigger
//https://snipboard.io/3WPmtJ.jpg

//This code step is always fired every time a story is created, updated or deleted.


//2. Identify_Stories - Node.js
//https://snipboard.io/drneKW.jpg

//This code step identifies whether the story is created or updated. If it is created, it must have the tag ‘add_to_Coda’. If it has the tag, the remaining code steps //will be performed (except Find_To_Be_Updated_Row) else, the program will be terminated. If the story is updated, the ‘add_to_Coda’ tag info is not available so next //code step ‘Find_To_Be_Updated_Row’ must be performed.

export default defineComponent({
  async run({ steps, $ }) {  
    console.log(steps);
    //console.log($);
   
    var found = 0;
    if (steps.trigger.event.body.actions[0].action == 'create'){
      for(var a in steps.trigger.event.body.references){            
        if(steps.trigger.event.body.references[a]['name'] == 'add_to_Coda'){
          found = 1;
        }
      }
      if(found==0){
        console.log('Not to be added.');
        return $.flow.exit();            
      }
      console.log('To be added.');
      // Means a new row is to be added.
      return 0;
    }
    if(steps.trigger.event.body.actions[0].action == 'update'){
      //console.log(steps.trigger.event.body.references);
      console.log('To be updated, if found in MJ');
      //Means a row is to be updated, if it exist.
      return 1;
    }  
   
  },
})



//3.Find_To_Be_Updated_Row - Python
//https://snipboard.io/AE3myi.jpg

//If the event is a ticket update, its rows must be in the MJ list. If not, then the next code steps will not be performed.


import requests
import json


def handler(pd: "pipedream"):


  #Ticket Update Event
  if pd.steps["Identify_Stories"]["$return_value"] == 1:
    print(json.dumps(pd.steps))
    headers = {'Authorization': 'Bearer d2bd31e7-8c46-40dd-be8d-e3dac91c498c'}
    story_number = str(pd.steps["trigger"]["event"]["body"]["primary_id"])
    print(story_number)


    uri = f'https://coda.io/apis/v1/docs/oBdvDQFaAf/tables/grid-BmMpHPf3fE/rows'  
    params = {
      'query': 'c-v8qep6COJs: ' + story_number,
    }
    req = requests.get(uri, headers=headers, params=params)
    req.raise_for_status() # Throw if there was an error.
    res = req.json()
    print(f'Matching rows: {len(res["items"])}')


    #Exit if a story number is not in the table.
    if len(res["items"]) < 1 :
      exit()
     
 
//4. Split_Title - Node.js
//https://snipboard.io/rSXU9w.jpg

//This code step tokenizes the title of the story into words. For example:

//Title : PA - UPC 2018
//Tokens : PA, - , UPC, 2018

//The first token will be the jurisdiction, the third token will be the adopted publication, and the 4th token will be the pub year. Title formatting must be uniform //for each shortcut stories, so that the title will be parsed correctly.

//5. JurisCode_to_JurisName - Node.js
//https://snipboard.io/i9AZkh.jpg

//This code step converts a jurisdiction abbreviation code to its actual jurisdiction name. For example: 

//AL => Alaska
//CA => California
//CA-LA = Los Angeles City

export default defineComponent({
  async run({ steps, $ }) {
    //Skip performing this function if the ticket is deleted.  
    if(steps.trigger.event.body.actions[0].action == 'delete')
      return


    //Converts a given jurisdiction code to its corresponding name.
    let juris_code = steps.Split_Title.$return_value[0];    
    let juris_array = [      
      ['AL', 'Alabama'],
      ['AK', 'Alaska'],
      ['AZ', 'Arizona'],
      ['AZ-PHX', 'Phoenix'],
      ['AR', 'Arkansas'],
      ['CA', 'California'],
      ['CA-LA', 'Los Angeles (City)'],
      ['CA-SF', 'San Francisco'],
      ['CO', 'Colorado'],
      ['CO-DEN', 'Denver'],
      ['CT', 'Connecticut'],
      ['DE', 'Delaware'],
      ['FL', 'Florida'],
      ['GA', 'Georgia'],
      ['HI', 'Hawaii'],
      ['ID', 'Idaho'],
      ['IL-CHI', 'Chicago'],
      ['IL-DPC', 'DuPage County'],
      ['IL', 'Illinois'],
      ['IL-SH', 'South Holland'],
      ['IN', 'Indiana'],
      ['IA', 'Iowa'],
      ['KS', 'Kansas'],
      ['KS-SDW', 'Wichita-Sedgwick'],
      ['KY', 'Kentucky'],
      ['LA', 'Louisiana'],
      ['ME', 'Maine'],
      ['MD', 'Maryland'],
      ['MA', 'Massachusetts'],
      ['MI-DET', 'Detroit'],
      ['MI', 'Michigan'],
      ['MN', 'Minnesota'],
      ['MS', 'Mississippi'],
      ['MO-KC', 'Kansas City'],
      ['MO', 'Missouri'],
      ['MT', 'Montana'],
      ['DC', 'District of Columbia'],
      ['PR', 'Puerto Rico'],
      ['NE', 'Nebraska'],
      ['NV-CLA', 'Clark County'],
      ['NV', 'Nevada'],
      ['NV-LV', 'Las Vegas'],
      ['NH', 'New Hampshire'],
      ['NJ', 'New Jersey'],
      ['NM', 'New Mexico'],
      ['NY', 'New York'],
      ['NY-NYC', 'New York City'],
      ['NC', 'North Carolina'],
      ['ND', 'North Dakota'],
      ['OH', 'Ohio'],
      ['OK', 'Oklahoma'],
      ['OR', 'Oregon'],
      ['OR-POR', 'Portland'],
      ['PA', 'Pennsylvania'],
      ['PA-PHI', 'Philadelphia'],
      ['RI', 'Rhode Island'],
      ['SC', 'South Carolina'],
      ['SD', 'South Dakota'],
      ['TN-KNX', 'Knoxville'],
      ['TN-NSH', 'Nashville and Davidson County'],
      ['TN', 'Tennessee'],
      ['TX-AU', 'Austin'],
      ['TX-DAL', 'Dallas'],
      ['TX-HOU', 'Houston'],
      ['TX-SA', 'San Antonio'],
      ['TX', 'Texas'],
      ['UT', 'Utah'],
      ['VT', 'Vermont'],
      ['VA', 'Virginia'],
      ['WA-SEA', 'Seattle'],
      ['WA', 'Washington'],
      ['WV', 'West Virginia'],
      ['WI', 'Wisconsin'],
      ['WY', 'Wyoming'],
     
    ];
    let juris_name = '';
    for (let a in juris_array){          
      if (juris_array[a][0] == juris_code){
        juris_name = juris_array[a][1];
      }
    }  
    return juris_name;    
  },
})


//6. Split_Description - Node.js
//https://snipboard.io/SCm06h.jpg

//This code step splits the body of the story. It looks for tags

//‘This is a code update’

//*Official Name:
//*Pub:
//*Agency:
//*Reference:
//*Citation:

//*Draft Doc Title:
//*Draft Source Doc:
//*Draft Drive:

//*Final Doc Title:
//*Final Source Doc:
//*Final Drive:

//*Effective Date:

//It uses regular expression, to match the tags and scan the text after the tag until the asterisk character *. The asterisk character * is added to parse tickets that //include multiple corresponding links for each tag. See Image 5.2. Code is modified so that tag sequences/order  will not matter. We can also delete any labels that //will not be of use(e.g. Draft Source doc and more)


Image 5.1
//https://snipboard.io/wTV4Ky.jpg

Image 5.2
//https://snipboard.io/HsDd8i.jpg

// To use previous step data, pass the `steps` object to the run() function
export default defineComponent({  
  async run({ steps, $ }){


    var description = '';
    var text_length = '';
    var description_updated = 1;


    if(steps.trigger.event.body.actions[0].action == 'delete')
      return  


    if (steps.trigger.event.body.actions[0].action == 'create'){      
      description = steps.trigger.event.body.actions[0].description;
      text_length = description.length;
    }else if (steps.trigger.event.body.actions[0].action == 'update'){
      try{
        //'Updated story description' event.        
        description = steps.trigger.event.body.actions[0].changes.description.new;      
        text_length = description.length;
      }catch(err){
        //'Updated story title' event.  
        description_updated = 0;            
      }      
    }
   
    if(description_updated == 1){
      console.log('Description:');
      console.log(description);


      //0.Extract Code Update
      var code_update = description.search("This is a code update");    
      if(code_update != -1){
        code_update = "true";
      }else{
        code_update = "false";
      }
      console.log('Code Update: ' + code_update);
     
      //1. Extract Official Name
      var official_name = description.search("Official Name");  
      if(official_name != -1){      
        official_name = description.match(/(?<=^\*\s*Official Name\s*:)(?:.(?!^\*))*/ms);              
        official_name = official_name?.[0].trim();                
        if(official_name=='None' || official_name == null){
          official_name = '';
        }
      }else{
        official_name = '';
      }
      console.log('Official Name: ' + official_name);


      //2.Extract Agency
      var agency = description.search("Agency");
      if(agency != -1){
        agency = description.match(/(?<=^\*\s*Agency\s*:)(?:.(?!^\*))*/ms);  
        agency = agency?.[0].trim();
        if(agency == null){
          agency= '';
        }
      }else{
        agency = '';
      }
      console.log('Agency: ' + agency);


      //3. Extract Reference    
      var reference = description.search("Reference");    
      if(reference != -1){        
        reference = description.match(/(?<=^\*\s*Reference\s*:)(?:.(?!^\*))*/ms);  
        reference  = reference?.[0].trim();
        if (reference == null){
          reference = '';
        }      
        reference = reference.split(/[\r\n]+/);                    
      }else{
        reference = '';
      }
      console.log('Reference:');
      console.log(reference);
           
      //4. Extract Citation
      var citation = description.search("Citation");
      if(citation != -1){
        citation = description.match(/(?<=^\*\s*Citation\s*:)(?:.(?!^\*))*/ms);  
        citation = citation?.[0].trim();
        if(citation == null){
          citation = '';
        }
      }else{
        citation = '';
      }
      console.log('Citation: ' + citation);


      //5. Extract Draft Doc Title
      var draft_doc_title = description.search("Draft Doc Title");
      if(draft_doc_title != -1){
        draft_doc_title = description.match(/(?<=^\*\s*Draft Doc Title\s*:)(?:.(?!^\*))*/ms);
        draft_doc_title = draft_doc_title?.[0].trim();
        if (draft_doc_title == null){
          draft_doc_title = '';
        }            
        draft_doc_title = draft_doc_title.split(/[\r\n]+/);
      }else{
        draft_doc_title = '';
      }
      console.log('Draft Doc Title: ');
      console.log(draft_doc_title);
 
      //6. Extract Draft Source Doc
      var draft_source_doc = description.search("Draft Source Doc");
      if(draft_source_doc != -1){
        draft_source_doc = description.match(/(?<=^\*\s*Draft Source Doc\s*:)(?:.(?!^\*))*/ms);
        draft_source_doc = draft_source_doc?.[0].trim();  
        if (draft_source_doc == null){
          draft_source_doc = '';
        }
        draft_source_doc = draft_source_doc.split(/[\r\n]+/);
      }else{
        draft_source_doc = '';
      }
      console.log('Draft Source Doc: ');
      console.log(draft_source_doc);


      //7. Extract Draft Drive
      var draft_drive = description.search("Draft Drive");
      if(draft_drive != -1){
        draft_drive = description.match(/(?<=^\*\s*Draft Drive\s*:)(?:.(?!^\*))*/ms);        
        draft_drive = draft_drive?.[0].trim();
        if (draft_drive == null){
          draft_drive = '';
        }
        draft_drive =  draft_drive.split(/[\r\n]+/);
      }else{
        draft_drive = '';
      }
      console.log('Draft Drive:' );
      console.log(draft_drive);


      //8. Extract Final Doc Title
      console.log('Final Doc Title:');
      var final_doc_title = description.search("Final Doc Title");
      if(final_doc_title != -1){    
        final_doc_title = description.match(/(?<=^\*\s*Final Doc Title\s*:)(?:.(?!^\*))*/ms);  
        final_doc_title = final_doc_title?.[0].trim();  
        if (final_doc_title == null){
          final_doc_title = '';
        }
        final_doc_title = final_doc_title.split(/[\r\n]+/);
      }else{
        final_doc_title = '';
      }
      console.log('Final Doc Title:');
      console.log(final_doc_title);


      //9. Extract Final Source Doc
      var final_source_doc = description.search("Final Source Doc");
      if(final_source_doc != -1){
        final_source_doc  = description.match(/(?<=^\*\s*Final Source Doc\s*:)(?:.(?!^\*))*/ms);  
        final_source_doc  = final_source_doc?.[0].trim();  
        if (final_source_doc == null){
          final_source_doc = '';
        }
        final_source_doc  = final_source_doc.split(/[\r\n]+/);
      }else{
        final_source_doc = '';
      }
      console.log('Final Source Doc: ');
      console.log(final_source_doc);


      //10. Extract Final Drive      
      var final_drive = description.search("Final Drive");
      if(final_drive != -1){
        final_drive = description.match(/(?<=^\*\s*Final Drive\s*:)(?:.(?!^\*))*/ms);  
        final_drive = final_drive?.[0].trim();
        if (final_drive == null){
          final_drive = '';
        }  
        final_drive = final_drive.split(/[\r\n]+/);
      }else{
        final_drive = '';
      }
      console.log('Final Drive:');
      console.log(final_drive);


      //11.Extract Effective Date:  
      var effective_date = description.search("Effective Date");
      if(effective_date != -1){
        effective_date = description.match(/(?<=^\*\s*Effective Date\s*:)(?:.(?!^\*))*/ms);  
        effective_date = effective_date?.[0].trim();
        if (effective_date  == null){
          effective_date = '';
        }  
      }else{
        effective_date = '';
      }
      console.log('Effective Date: ' + effective_date);  
    }
   
    //12. Reviewed Entry
    var reviewed_entry = "false";
   
    return [code_update, official_name, agency, reference, citation, draft_doc_title, draft_source_doc, draft_drive, final_doc_title, final_source_doc, final_drive, effective_date, reviewed_entry, description_updated]
  },
})



//7. Prepare_Coda_Row_Entries - Node.js
//https://snipboard.io/ElYQNL.jpg

//This code step is important, especially if it includes multiple corresponding links for each tag. See Image 5.2. It formats data into multiple rows ready for //insertion/update  to Coda. For Image 5.2, it will create two rows with story number 45455 and 45455-2. This is the code logic.

//It will scan the links corresponding to tags ‘Reference’, ‘Draft Doc Title’, ‘Draft Source Doc’, ’Draft Drive’, ‘Final Doc Title’, ‘Final Source Doc’ and ‘Final //Drive’. For example, below are the number of links found right after each tag.

//Reference - 1
//Draft Doc Title - 2 
//Draft Source Doc - 2
//Draft Drive - 2
//Final Doc Title -3
//Final Source Doc -3 
//Final Drive -3

//Since the maximum number of links found for each tag is 3, then it will create 3 rows. Their story numbers will be: {story_number},  {story_number-2},  //{story_number-3}]

//In the example, each row will include the following data.

//{story_number}-> Reference 1, Draft Doc Title 1, Draft Source Doc 1, Draft Drive 1, Final Doc Title 1, Final  Source Doc 1, Final  Drive 1 and other info (e.g //code_update, official name, agency, citation and effective date )

//{story_number-2}-> Draft Doc Title 2, Draft Source Doc 2, Draft Drive 2, Final Doc Title 2, Final  Source Doc 2, Final  Drive 2 and other info (e.g code_update, //official name, agency, citation and effective date )

//{story_number-3}-> Final Doc Title 3, Final  Source Doc 3, Final  Drive 3 and other info (e.g code_update, official name, agency, citation and effective date )

// To use previous step data, pass the `steps` object to the run() function
export default defineComponent({
  async run({ steps, $ }) {


    if(steps.trigger.event.body.actions[0].action == 'delete')
      return  
       
    // Return data to use it in future steps
    //console.log(steps.Split_Description.$return_value);
    var multiple = 0;
    var multiple_array = [];
    var description_updated = steps.Split_Description.$return_value[13];
   
    //Story description is updated.
    if(description_updated == 1){
      console.log('Nisulod');


      console.log('Description Updated ');
      var reference_count = steps.Split_Description.$return_value[3].length;
      if(reference_count > multiple){
        multiple = reference_count -1;
      }      
     
      console.log('Reference Count');
      console.log(reference_count);


   
      var draft_doc_title_count = steps.Split_Description.$return_value[5].length;
      if(draft_doc_title_count > multiple){
        multiple = draft_doc_title_count - 1;
      }
     


      var draft_source_doc_count = steps.Split_Description.$return_value[6].length;
      if(draft_source_doc_count > multiple){
        multiple = draft_source_doc_count;
      }
     


      var draft_drive_count = steps.Split_Description.$return_value[7].length;      
      if(draft_drive_count > multiple){
        multiple = draft_drive_count;
      }


      var final_doc_title_count = steps.Split_Description.$return_value[8].length;
      if(final_doc_title_count > multiple){
        multiple = final_doc_title_count;
      }
     


      var final_source_doc_count = steps.Split_Description.$return_value[9].length;
      if(final_source_doc_count  > multiple){
        multiple = final_source_doc_count ;
      }


      var final_drive_count = steps.Split_Description.$return_value[10].length;  
      if(final_drive_count   > multiple){
        multiple = final_drive_count  ;
      }
      console.log(multiple);
   
      for (var i = 0; i < multiple; i++){  
           
        multiple_array[i] = [];
        multiple_array[i][0] = steps.Split_Description.$return_value[0];        
        multiple_array[i][1] = steps.Split_Description.$return_value[1];        
        multiple_array[i][2] = steps.Split_Description.$return_value[2];
       
        if(i < reference_count){
          multiple_array[i][3] = steps.Split_Description.$return_value[3][i];
        }else{
          multiple_array[i][3] = '';
        }
       
        multiple_array[i][4] = steps.Split_Description.$return_value[4];
     
        if(i < draft_doc_title_count){
          multiple_array[i][5] = steps.Split_Description.$return_value[5][i];
        }else{
          multiple_array[i][5] = '';
        }
       
        if(i < draft_source_doc_count){
          multiple_array[i][6] = steps.Split_Description.$return_value[6][i];
        }else{
          multiple_array[i][6] = '';
        }
     
        if(i < draft_drive_count){
          multiple_array[i][7] = steps.Split_Description.$return_value[7][i];
        }else{
          multiple_array[i][7] = '';
        }
       
        if(i < final_doc_title_count){
          multiple_array[i][8] = steps.Split_Description.$return_value[8][i];
        }else{
          multiple_array[i][8] = '';
        }
       
        if(i < final_source_doc_count){
          multiple_array[i][9] = steps.Split_Description.$return_value[9][i];
        }else{
          multiple_array[i][9] = '';
        }
       
        if(i < final_drive_count){
          multiple_array[i][10] = steps.Split_Description.$return_value[10][i];
        }else{
          multiple_array[i][10] = '';
        }


        //Effective Date      
        multiple_array[i][11] = steps.Split_Description.$return_value[11];
       
        //Story Number
        if(i==0){
          multiple_array[i][12] = steps.trigger.event.body.primary_id;
        }else{
          multiple_array[i][12] = steps.trigger.event.body.primary_id + '-' + (i+1);
        }
             
        //Jurisdiction
        multiple_array[i][13] = steps.JurisCode_to_JurisName.$return_value
       
        //Publication
        multiple_array[i][14] = steps.Split_Title.$return_value[2];
        if(multiple_array[i][14]== null){
          multiple_array[i][14] = '';
        }


        //Publication Year
        multiple_array[i][15] = steps.Split_Title.$return_value[3];
        if(multiple_array[i][15]== null){
          multiple_array[i][15] = '';
        }


        //Reviewed Entry
        multiple_array[i][16] = 'false';        
      }
    }else{
      console.log('Description Not Updated ');
    }
   
    return multiple_array;
  },
})

//8. Upsert_Coda_Entries - Python
//https://snipboard.io/fe6Uvk.jpg

//This code step, inserts/update the parsed data to Coda table ‘Auto-CodeBook’. Every trigger event, we need to perform multiple insertion, update or query(specially //for sources with multiple links).  However, Pipedream has a limitation of not being able to perform a code step multiple times. See this link. One workaround to this //limitation is to invoke another HTTP requests which may consume multiple credits for each trigger event. This is very expensive since Pipedream counts the number of //credits for a price. Thus, the pre-built code step action of upserting to Coda is rewritten to a Python code to minimize the credit usage to only 1 per trigger //event.

import requests


def handler(pd: "pipedream"):  
  if(pd.steps["trigger"]["event"]["body"]["actions"][0]['action'] == 'delete'):
    return
   
  headers = {'Authorization': 'Bearer bfa5b51b-1494-438e-b3e8-ba725f94cf8f'}
  number_of_entries = len(pd.steps["Prepare_Coda_Row_Entries"]['$return_value'])
  updated_story_number = '';
  description_updated = pd.steps["Split_Description"]["$return_value"][13];
  insert = 0
 
  if description_updated==1: #Ticket Desciption is updated.
    loop = 0
    while loop < number_of_entries:
      if (pd.steps["trigger"]["event"]["body"]["actions"][0]["action"] == 'create' or insert == 1):  
        #Insert Rows
        uri = f'https://coda.io/apis/v1/docs/oBdvDQFaAf/tables/grid-BmMpHPf3fE/rows'
        payload = {
          'rows': [
            {
              'cells': [
                {'column': 'c-4gFd54jyqu', 'value': pd.steps["Prepare_Coda_Row_Entries"]['$return_value'][loop][0]},   #code_update
                {'column': 'c-vyu-tblRaR', 'value': pd.steps["Prepare_Coda_Row_Entries"]['$return_value'][loop][1]},   #official_name
                {'column': 'c-uVaO8dOdKA', 'value': pd.steps["Prepare_Coda_Row_Entries"]['$return_value'][loop][2]},   #agency
                {'column': 'c-ZBgg_QabFq', 'value': pd.steps["Prepare_Coda_Row_Entries"]['$return_value'][loop][3]},   #reference
                {'column': 'c-IX7TjjEpnx', 'value': pd.steps["Prepare_Coda_Row_Entries"]['$return_value'][loop][4]},   #citation
                {'column': 'c-WoyiQ55JUN', 'value': pd.steps["Prepare_Coda_Row_Entries"]['$return_value'][loop][5]},   #draft_doc_title
                {'column': 'c-LFkFq2v6jU', 'value': pd.steps["Prepare_Coda_Row_Entries"]['$return_value'][loop][6]},   #draft_source_doc
                {'column': 'c-9BYtgrhATR', 'value': pd.steps["Prepare_Coda_Row_Entries"]['$return_value'][loop][7]},   #draft_drive
                {'column': 'c-u-ANu12hSD', 'value': pd.steps["Prepare_Coda_Row_Entries"]['$return_value'][loop][8]},   #final_doc_title
                {'column': 'c-MUAYBkxZsM', 'value': pd.steps["Prepare_Coda_Row_Entries"]['$return_value'][loop][9]},   #final_source_doc
                {'column': 'c-l2tsysrunU', 'value': pd.steps["Prepare_Coda_Row_Entries"]['$return_value'][loop][10]},  #final_drive
                {'column': 'c-mKi5RZW0a9', 'value': pd.steps["Prepare_Coda_Row_Entries"]['$return_value'][loop][11]},  #effective_date
                {'column': 'c-v8qep6COJs', 'value': pd.steps["Prepare_Coda_Row_Entries"]['$return_value'][loop][12]},  #story_number
                {'column': 'c-GNm2sAnKWs', 'value': pd.steps["Prepare_Coda_Row_Entries"]['$return_value'][loop][13]},  #jurisdiction
                {'column': 'c-WEylFKks_C', 'value': pd.steps["Prepare_Coda_Row_Entries"]['$return_value'][loop][14]},  #publication
                {'column': 'c-AeOUqPm1iy', 'value': pd.steps["Prepare_Coda_Row_Entries"]['$return_value'][loop][15]},  #publication_year
                {'column': 'c-CR9mEEvFII', 'value': pd.steps["Prepare_Coda_Row_Entries"]['$return_value'][loop][16]}   #reviewed_entry
              ],
            },
          ],
        }
      
        req = requests.post(uri, headers=headers, json=payload)


      else:


        #Find Row ID's
        uri = f'https://coda.io/apis/v1/docs/oBdvDQFaAf/tables/grid-BmMpHPf3fE/rows'      
        params = {
          'query': 'c-v8qep6COJs:"' +  str(pd.steps["Prepare_Coda_Row_Entries"]['$return_value'][loop][12]) + '"',
        }
     
        print(pd.steps["Prepare_Coda_Row_Entries"]['$return_value'][loop][12])
        req = requests.get(uri, headers=headers, params=params)
        req.raise_for_status() # Throw if there was an error.
        res = req.json()    


        try:  
          updated_story_number = res["items"][0]['id']
        except:
          insert = 1
          continue


        print(f'Matching rows: {res["items"][0]["id"]}')


        #Update Row
        uri = f'https://coda.io/apis/v1/docs/oBdvDQFaAf/tables/grid-BmMpHPf3fE/rows/' + updated_story_number
         
        payload = {
          'row': {
            'cells': [
              {'column': 'c-4gFd54jyqu', 'value': pd.steps["Prepare_Coda_Row_Entries"]['$return_value'][loop][0]},   #code_update
              {'column': 'c-vyu-tblRaR', 'value': pd.steps["Prepare_Coda_Row_Entries"]['$return_value'][loop][1]},   #official_name
              {'column': 'c-uVaO8dOdKA', 'value': pd.steps["Prepare_Coda_Row_Entries"]['$return_value'][loop][2]},   #agency
              {'column': 'c-ZBgg_QabFq', 'value': pd.steps["Prepare_Coda_Row_Entries"]['$return_value'][loop][3]},   #reference
              {'column': 'c-IX7TjjEpnx', 'value': pd.steps["Prepare_Coda_Row_Entries"]['$return_value'][loop][4]},   #citation
              {'column': 'c-WoyiQ55JUN', 'value': pd.steps["Prepare_Coda_Row_Entries"]['$return_value'][loop][5]},   #draft_doc_title
              {'column': 'c-LFkFq2v6jU', 'value': pd.steps["Prepare_Coda_Row_Entries"]['$return_value'][loop][6]},   #draft_source_doc
              {'column': 'c-9BYtgrhATR', 'value': pd.steps["Prepare_Coda_Row_Entries"]['$return_value'][loop][7]},   #draft_drive
              {'column': 'c-u-ANu12hSD', 'value': pd.steps["Prepare_Coda_Row_Entries"]['$return_value'][loop][8]},   #final_doc_title
              {'column': 'c-MUAYBkxZsM', 'value': pd.steps["Prepare_Coda_Row_Entries"]['$return_value'][loop][9]},   #final_source_doc
              {'column': 'c-l2tsysrunU', 'value': pd.steps["Prepare_Coda_Row_Entries"]['$return_value'][loop][10]},  #final_drive
              {'column': 'c-mKi5RZW0a9', 'value': pd.steps["Prepare_Coda_Row_Entries"]['$return_value'][loop][11]},  #effective_date
              {'column': 'c-v8qep6COJs', 'value': pd.steps["Prepare_Coda_Row_Entries"]['$return_value'][loop][12]},  #story_number
              {'column': 'c-GNm2sAnKWs', 'value': pd.steps["Prepare_Coda_Row_Entries"]['$return_value'][loop][13]},  #jurisdiction
              {'column': 'c-WEylFKks_C', 'value': pd.steps["Prepare_Coda_Row_Entries"]['$return_value'][loop][14]},  #publication
              {'column': 'c-AeOUqPm1iy', 'value': pd.steps["Prepare_Coda_Row_Entries"]['$return_value'][loop][15]},  #publication_year
              {'column': 'c-CR9mEEvFII', 'value': pd.steps["Prepare_Coda_Row_Entries"]['$return_value'][loop][16]}   #reviewed_entry
            ],
          },      
        }    
        req = requests.put(uri, headers=headers, json=payload)


      req.raise_for_status() # Throw if there was an error.
      res = req.json()
   
      if (pd.steps["trigger"]["event"]["body"]["actions"][0]["action"] == 'create' or insert == 1):
        print(f'Successful row insertion.')
      else:
        print(f'Updated row: {updated_story_number}')      
      loop = loop + 1


  else: #Ticket title is updated.
    base_point = 0
    while True:
     
      if base_point == 0:
        story_number = str(pd.steps["trigger"]["event"]["body"]["primary_id"])
      else:
        story_number = str(pd.steps["trigger"]["event"]["body"]["primary_id"]) + '-' + str(base_point + 1)


      print('Updating title for : ' + story_number)
      headers = {'Authorization': 'Bearer bfa5b51b-1494-438e-b3e8-ba725f94cf8f'}
      uri = f'https://coda.io/apis/v1/docs/oBdvDQFaAf/tables/grid-BmMpHPf3fE/rows'
   
      params = {
        'query': 'c-v8qep6COJs:"' + story_number + '"',
      }
      req = requests.get(uri, headers=headers, params=params)
      req.raise_for_status() # Throw if there was an error.
      res = req.json()


      print(f'Matching rows: {len(res["items"])}')
      result = len(res["items"])


      if result > 0 :
        uri = f'https://coda.io/apis/v1/docs/oBdvDQFaAf/tables/grid-BmMpHPf3fE/rows/' + res["items"][0]["id"]
        payload = {
          'row': {
            'cells': [
              {'column': 'c-GNm2sAnKWs', 'value': pd.steps["JurisCode_to_JurisName"]["$return_value"]},  #Update Jurisdiction
              {'column': 'c-WEylFKks_C', 'value': pd.steps["Split_Title"]["$return_value"][2]},  #Update Publication
              {'column': 'c-AeOUqPm1iy', 'value': pd.steps["Split_Title"]["$return_value"][3]},  #Update Publication Year
              {'column': 'c-CR9mEEvFII', 'value': 'false'} #Update Reviewed Entry
            ],
          },
        }
        req = requests.put(uri, headers=headers, json=payload)
        req.raise_for_status() # Throw if there was an error.
   
        print(f'Updated row {story_number}')  
      else:
        break


      base_point = base_point + 1
     
 

//9. Mark_Rows_For_Deletion  - Python
//https://snipboard.io/hVTjuB.jpg

//This code step mark rows for deletion. There are instances that rows in Coda needs to be deleted.  Manual review is needed for deleting row in Coda to avoid data //loss. For example:

//On ticket creation, 3 links are added to the tag ‘Final Source:’. Thus, it will create 3 rows in Coda.
//1.If upon ticket update, it is modified to have two links only, the third row will be then marked for deletion.

//2.If a story is deleted in Shortcut.
//https://snipboard.io/eGwg5c.jpg

import requests

def handler(pd: "pipedream"):
   
  story_deleted = 0
  if(pd.steps["trigger"]["event"]["body"]["actions"][0]['action'] == 'delete'):
    story_deleted = 1
    base_point = 1
  else:
    base_point = len(pd.steps["Prepare_Coda_Row_Entries"]['$return_value']) + 1;
   
    description_updated = pd.steps["Split_Description"]["$return_value"][13];
    if (description_updated == 0):    
      return
         
    print(pd.steps["Split_Description"])
 
  while True:
    if (base_point == 1):
      story_number = str(pd.steps["trigger"]["event"]["body"]["primary_id"])
    else:
      story_number = str(pd.steps["trigger"]["event"]["body"]["primary_id"]) + '-' + str(base_point)
 
    print('Searching for : ' + story_number)
    headers = {'Authorization': 'Bearer bfa5b51b-1494-438e-b3e8-ba725f94cf8f'}
    uri = f'https://coda.io/apis/v1/docs/oBdvDQFaAf/tables/grid-BmMpHPf3fE/rows'
   
    params = {
      'query': 'c-v8qep6COJs:"' + story_number + '"',
    }
    req = requests.get(uri, headers=headers, params=params)
    req.raise_for_status() # Throw if there was an error.
    res = req.json()


    print(f'Matching rows: {len(res["items"])}')
    result = len(res["items"])


    if(result > 0):
      while(result > 0):  
        print('Marking row : ' +  res["items"][result-1]["id"])
        uri = f'https://coda.io/apis/v1/docs/oBdvDQFaAf/tables/grid-BmMpHPf3fE/rows/' + res["items"][result-1]["id"]
        status = 1
        if(story_deleted == 1):
          status = 'Ticket Deleted'
        else:
          status = 'Ticket Updated'


        payload = {
          'row': {
            'cells': [
              {'column': 'c-ipfrmFlNn0', 'value': 'For Deletion : ' + status},  #Update column Review Notes
              {'column': 'c-CR9mEEvFII', 'value': 'false'} #Update Reviewed Entry
            ],
          },
        }
        req = requests.put(uri, headers=headers, json=payload)
        req.raise_for_status() # Throw if there was an error.
   
        print(f'Updated row {res["items"][result-1]["id"]}')  
        result = result - 1
      base_point = base_point + 1
    else:
      break
 
//Everything that is newly inserted, updated and to be deleted will have an uncheck value for “Reviewed Entry” column, so that we can still verify the correctness of //data, if there is any. Also, there are columns that still needs to be manually entered like below:

//https://snipboard.io/PnNrwC.jpg
























