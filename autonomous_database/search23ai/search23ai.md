# Integrated Approaches to Searching and Analyzing Data in Oracle Database 23ai

## Introduction

Integrated Approaches to Searching and Analyzing Data in Oracle Database 23ai

Oracle Database 23ai introduces groundbreaking capabilities that make searching and analyzing data faster, smarter, and more intuitive than ever before. In this session, we’ll explore the powerful new features that bring AI and advanced search together, including:

* Ubiquitous Search
* Vector Search
* Natural Language to SQL
* Document Search
* Hybrid Search
* Integrated examples that combine the above techniques

You’ll learn when to use each method, see each one in action, and discover how they can be combined to solve real-world data challenges—whether you're building intelligent apps, working with unstructured data, or enabling natural language interactions.

[Integrated Approaches to Searching and Analyzing Data in Oracle Database 23ai](youtube:VudVfg8sXwY)

## New to the OCI Generative AI Service

Please reference the below links for more information on Oracle's Artificial Intelligence (AI) and Generative AI Offerings.

* [Artificial Intelligence (AI)](https://www.oracle.com/artificial-intelligence/)

* [Generative AI capabilities](https://www.oracle.com/artificial-intelligence/generative-ai/)

* [Generative AI Service](https://www.oracle.com/artificial-intelligence/generative-ai/generative-ai-service/)

* [Pretrained Foundational Models](https://docs.oracle.com/en-us/iaas/Content/generative-ai/pretrained-models.htm)

## Documentation Links

* [Oracle AI Vector Search User's Guide](https://docs.oracle.com/en/database/oracle/oracle-database/23/vecse/index.html)

* [Perform Hybrid Search](https://docs.oracle.com/en/database/oracle/oracle-database/23/vecse/perform-hybrid-search.html)

* [Select AI for ADB](https://docs.oracle.com/en-us/iaas/autonomous-database-serverless/doc/sql-generation-ai-autonomous.html)

* [Oracle Database 23ai](https://docs.oracle.com/en/database/oracle/oracle-database/23/index.html)

* [Oracle Autonomous Database Serverless](https://docs.oracle.com/en/cloud/paas/autonomous-database/serverless/index.html)

## Cloud Coaching Webinar Playlist

1. Cloud Coaching YouTube Playlist

	* [Code Innovate Cloud Coaching Playlist](https://www.youtube.com/playlist?list=PLPIzp-E1msrZMCfSHbKgLK3KWsNM9JB9a)

## Prerequisites: 

1.  Autonomous Database Version 23ai

2.  Access to the OCI GenAI Services in a Supported Region (see Task 0 for Details)

3.  Access to OCI Object Storage (see Task 0 for Details)

## Task 0: PLEASE READ - Verify Access to OCI Gen AI Service and Object Storage

The OCI Gen AI Service is currenly available in specific OCI Regions.  If you currently are **NOT** subscribed to one of those regions, you must do so to access the OCI Gen AI Service and leverage the pretrained models hosted by the service.  You must also ensure the identity domain is being replicated to the region where you plan to access the OCI Gen AI Services.  Lastly, you will also need to setup IAM Policies so the users can access the OCI Gen AI Service.  Please see the links below for all those details.

1. OCI Gen AI Service availabilty

	* [Regions with Generative AI](https://docs.oracle.com/en-us/iaas/Content/generative-ai/overview.htm#regions)

2. Managing and Subscribing to a Region (optional)

	* [Managing Regions](https://docs.oracle.com/en-us/iaas/Content/Identity/regions/managingregions.htm)

	* [Subscribing to an Infrastructure Region](https://docs.oracle.com/en-us/iaas/Content/Identity/regions/To_subscribe_to_an_infrastructure_region.htm#subscribe)
	
3. Enable Replicating an Identity Domain to Multiple Regions (optional based on step 2)

	* [Replicating an Identity Domain to Multiple Regions](https://docs.oracle.com/en-us/iaas/Content/Identity/domains/to-manage-regions-for-domains.htm)

4. Setup IAM Policies to Access the OCI Gen AI resources

	* [Getting Access to Generative AI](https://docs.oracle.com/en-us/iaas/Content/generative-ai/iam-policies.htm)

5. Setup IAM Policies to Access the OCI Object Storage

	* [Using Object Storage](https://docs.oracle.com/en-us/iaas/Content/Object/Concepts/objectstorageoverview.htm#using)

	* [Managing Access to Object Storage](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/objectstoragepolicyreference.htm)

## Task 1: Create Database User with Grants

As the user **ADMIN**, issue the below SQL Statements

1. Create Database User and Initial Grants

    ```
    <copy>
    create user search23ai identified by {password};
    grant dwrole to search23ai;
    grant unlimited tablespace to search23ai;
    grant create job to search23ai;
    </copy>
    ```

2. Grant Database User Access to DBMS Packages

    ```
    <copy>
    grant execute on DBMS_CLOUD to search23ai;
    grant execute on DBMS_VECTOR to search23ai;
    grant execute on dbms_cloud_ai to search23ai;   
    </copy>
    ```

3.  Grant Database User Access to Database Actions - SQL Developer Web 

    ```
    <copy>
    BEGIN
     ords_admin.enable_schema(
     p_enabled => TRUE,
     p_schema => 'search23ai',
     p_url_mapping_type => 'BASE_PATH',
     p_url_mapping_pattern => 'search23ai',
     p_auto_rest_auth => NULL
     );
     commit;
    END;
    /
    </copy>
    ```

## Task 2: Update Access Control List

As the user **ADMIN**, issue the following PL/SQL Code

1. Grant Non-Admin User Permission to Access OCI Gen AI Provider

    ```
    <copy>
    BEGIN
    DBMS_NETWORK_ACL_ADMIN.APPEND_HOST_ACE (
      HOST         => 'inference.generativeai.{region}.oci.oraclecloud.com',
      ACE          => xs$ace_type(
          PRIVILEGE_LIST => xs$name_list('http'),
          PRINCIPAL_NAME => 'search23ai',
          PRINCIPAL_TYPE => xs_acl.ptype_db));
    END;
    /
    </copy>
    ```

2. Grant Non-Admin User Permission to Access Cohere AI Provider

    ```
    <copy>
    BEGIN
    DBMS_NETWORK_ACL_ADMIN.APPEND_HOST_ACE (
      HOST         => 'api.cohere.com',
      LOWER_PORT   => 443,
      UPPER_PORT   => 443,
      ACE          => xs$ace_type(
          PRIVILEGE_LIST => xs$name_list('http'),
          PRINCIPAL_NAME => 'search23ai',
          PRINCIPAL_TYPE => xs_acl.ptype_db));
    END;
    /
    </copy>
    ```

3. View Existing ACL Priviledges 

    ```
    <copy>
	select a.host, b.principal, b.privilege, b.is_grant 
    from dba_network_acls a, dba_network_acl_privileges b
    where a.acl = b.acl;     
    </copy>
    ```

## Task 3: Setup Moviestream Data for Loading

As the user **ADMIN**, issue the below PL/SQL Code.

1. Execute the Moviestream Setup Script

    ```
    <copy>
    declare 
        l_uri varchar2(500) := 'https://objectstorage.us-ashburn-1.oraclecloud.com/n/c4u04/b/building_blocks_utilities/o/setup/workshop-setup.sql';
    begin
        dbms_cloud_repo.install_sql(
            content => to_clob(dbms_cloud.get_object(object_uri => l_uri))
        );
    end;
    /
    </copy>
    ```

2. Add Moviestream Grants to Search23ai User 

    ```
    <copy>
    begin
        workshop.write('Begin demo install');
        workshop.write('add user search23ai', 1);
        workshop.add_adb_user('search23ai','{password}');
    end;
    /
    </copy>
    ```

## Task 4: Create and Test Credentials

As the user **SEARCH23AI**, issue the below PL/SQL Code.

1. Create API Credential to Access OCI Gen AI Service

    ```
    <copy>
	BEGIN
	   DBMS_CLOUD.CREATE_CREDENTIAL (
		   credential_name => '{enter_credential_name}',
		   user_ocid       => 'ocid1.user.oc1......',
		   tenancy_ocid    => 'ocid1.tenancy.oc1......',
		   private_key     => '-----BEGIN PRIVATE KEY-----
							   M..................
							   -----END PRIVATE KEY-----',
		   fingerprint     => '3e:...............:ee');
	END;
	/
    </copy>
    ```

    For more details, see the [`DBMS_CLOUD.CREATE_CREDENTIAL` Procedure](https://docs.oracle.com/en/cloud/paas/autonomous-database/serverless/adbsb/dbms-cloud-subprograms.html#GUID-742FC365-AA09-48A8-922C-1987795CF36A) documentation.  

	For more details on API Signing Keys, see the [Required Keys and OCIDs](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/apisigningkey.htm) documentation.

    If you would like to see what existing credentials exist, execute the following SQL Statement

    ```
    <copy>
    select * from user_credentials;  
    </copy>
    ```

2. Test OCI API Credential

	This test connectivity to OCI Object Storage.

    ```
    <copy>
	SELECT * FROM DBMS_CLOUD.LIST_OBJECTS('{oci_api_cred_from_Task_4_1}','https://objectstorage.{region}.oraclecloud.com/n/{namespace}/b/{bucket_name}/o/');
    </copy>
    ```

	This test connectivity to OCI Gen AI Service.

    ```
    <copy>
	set serveroutput on;
	DECLARE
		-- https://docs.oracle.com/en-us/iaas/Content/generative-ai/pretrained-models.htm
		gen_ai_endpoint 	varchar2(500) := 'https://inference.generativeai.{region}.oci.oraclecloud.com';
		gen_ai_model 		varchar2(500) := 'cohere.command-r-08-2024'; --cohere.command-r-plus-08-2024
		compartment_ocid	varchar2(500) := '{update with your compartment ocid};
		api_cred_name 		varchar2(500) := '{oci_api_cred_from_Task_4_1}';
		ai_prompt 			varchar2(4000) := 'who is Babe Ruth?';
		resp 				dbms_cloud_types.RESP;
	BEGIN
		resp := dbms_cloud.send_request(
			credential_name => api_cred_name,
			uri => gen_ai_endpoint || '/20231130/actions/chat',
			method => dbms_cloud.METHOD_POST,
			body => utl_raw.cast_to_raw(json_object(
				'compartmentId'     value compartment_ocid,
				'servingMode'       value
					(json_object(
						'modelId'               value gen_ai_model,
						'servingType'           value 'ON_DEMAND'
						)),
						'chatRequest'           value 
						(json_object(
							'message'           value ai_prompt,
							'apiFormat'         value 'COHERE',
							'maxTokens'         value 2000,
							'temperature'       value 0.75,
							'frequencyPenalty'  value 0,
							'presencePenalty'   value 0,
							'topP'              value 1.0,
							'topK'              value 0,
							'isStream'          value false
						))
			))
		);
		dbms_output.put_line(dbms_cloud.get_response_text(resp)); 
	END;
	/
	</copy>
    ```

3. Create Vector Credential to Perform Vector Tasks

	For more details, see the [`DBMS_VECTOR.CREATE_CREDENTIAL` Procedure](https://docs.oracle.com/en/database//oracle/oracle-database/23/arpls/dbms_vector1.html#GUID-4BBCBF46-3903-4EBB-8BE8-A7690151CF25) documentation.

	**Note** - You pass the <private key string> value (excluding the BEGIN and END lines in the key file), either as a single line or as multiple lines.

    ```
    <copy>
	declare
	  jo json_object_t;
	begin
	  jo := json_object_t();
	  jo.put('user_ocid','ocid1.user.oc1......');
	  jo.put('tenancy_ocid','ocid1.tenancy.oc1......');
	  jo.put('compartment_ocid','ocid1.compartment.oc1.......');
	  jo.put('private_key','M------------------');
	  jo.put('fingerprint','3e:...............:ee');
	  dbms_output.put_line(jo.to_string);
	  dbms_vector.create_credential(
		credential_name   => '{enter_credential_name}',
		params            => json(jo.to_string));
	end;
	/	
    </copy>
    ```

4. Test Vector Credential

    ```
    <copy>
	select dbms_vector_chain.utl_to_embedding('embed some text', json('{
	  "provider": "OCIGenAI",
	  "credential_name": "{oci_vector_cred_from_Task_4_3}",
	  "url": "https://inference.generativeai.{region}.oci.oraclecloud.com/20231130/actions/embedText",
	  "model": "cohere.embed-english-v3.0",
	  "batch_size":10
	}')) embed_vector
	from dual;	
    </copy>
    ```
    
5. Create Cohere Credential - used for re-ranking
    
    ```
    <copy>
	BEGIN
	  DBMS_CLOUD.CREATE_CREDENTIAL(
		credential_name => 'COHERE_CRED',
		username => '<cohere username>',
		password => '<cohere password>');
	END;
	/
    </copy>
    ```

	**Note** - The **username** is the email address you used to log into https://cohere.com/.  The **password** represents a trial key accessible from the **API Keys** section.

## Task 5: Create Database Objects - Tables, Views, Packages, Trigger, Sequences, Select AI Profiles

As the user **SEARCH23AI**, issue the below SQL Code.

**Note** - Be sure to update the SQL Code accordingly where you see **{....}**

1. Create Documents Tables

    ```
    <copy>
    Create table documents 
       (id number(*,0) generated by default on null as identity minvalue 1 maxvalue 9999999999999999999999999999 increment by 1 start with 1 cache 20 noorder  nocycle  nokeep  noscale  not null enable, 
        file_name varchar2(900 byte), 
        file_size number(*,0), 
        file_type varchar2(100 byte) default 'application/pdf', 
        file_content blob, 
        first_page_image blob, 
        mimetype varchar2(50 byte),
        domain varchar2(100) default 'Default',
        file_content_clob clob, 
        primary key (id)
    );
    </copy>
    ```
    
2. Create Documents Vector Table

    ```
    create table document_vector 
     (id number(*,0) not null enable, 
      chunk_id number, 
      chunk_pos number, 
      chunk_size number, 
      chunk_txt varchar2(4000 byte), 
      embed_vector vector, 
       constraint fk_id foreign key (id)
       references documents (id) on delete cascade enable
    );
    ```

3. Create Conversations Table

    ```
    <copy>
    create table conversations 
       (id number, 
        username varchar2(100 char), 
        started_on timestamp (6), 
        app_session number, 
         constraint conversations_id_pk primary key (id)
        );
    </copy>
    ```

4. Create Prompts Table

    ```
    <copy>
    create table prompts
       (id number, 
        conv_id number not null enable, 
        prompt varchar2(32767 byte), 
        response varchar2(32767 byte), 
        asked_on timestamp (6), 
        chathistory varchar2(32767 byte),
        references varchar2(32767 byte), 
        request varchar2(32767 byte), 
        output varchar2(32767 byte), 
        citations varchar2(32767 byte), 
        documents varchar2(32767 byte), 
        documents4rerank varchar2(32767 byte), 
        documentsreranked varchar2(32767 byte),
        showsql varchar2(32767),
        ai_service varchar2(100 byte),
        domain varchar2(100 byte),
        constraint prompts_id_pk2 primary key (id),
        foreign key (conv_id) references conversations (id)
       );
    </copy>
    ```

5. Create Prompt_Doc Table

    ```
    <copy>
    create table prompt_docs 
       (document_id number, 
        conv_id number, 
        prompt_id number, 
        chunk_id number, 
        index_id number, 
        chunk varchar2(32767 byte)
       );
    </copy>
    ```
6. Create Prompt Instructions Table and Seed Data

    ```
    <copy>
    create table prompt_instructions    
       (id number(*,0) generated by default on null as identity minvalue 1 maxvalue 9999999999999999999999999999 increment by 1 start with 1 cache 20 noorder  nocycle  nokeep  noscale  not null enable, 
        instruction varchar2(32767), 
        active char(1),
        primary key (id)
        );

    insert into prompt_instructions values(1,'Question: ','Y');
    commit;
    </copy>
    ```

7. Create AI Configuration Table and Seed Data

    ```
    <copy>
    create table ai_configuration(
        id                        number,
        llm_max_tokens          number,
        llm_temperature         number,
        llm_frequencypenalty    number,
        llm_presencepenalty     number,
        llm_topp                number,
        llm_topk                number,
        vector_by               varchar2(100),
        vector_max              number,
        vector_overlap          number,
        vector_split            varchar2(100),
        vector_language         varchar2(100),
        vector_normalize        varchar2(100),
        top_doc_chunks          number);

    insert into ai_configuration values(1,4000,0.1,0,0,1,0,'words',100,0,'sentence','american','all',5);
    commit;
    </copy>
    ```

8. Create Domains Table and Seed Data

    ```
    <copy>
    create table domains(
        domain_name varchar2(100),
        ai_service varchar2(100),
        domain_description varchar2(4000),
        ai_profile varchar2(100));

    insert into domains values ('Default','Vector Search','Vector search of documents related to Texas house bills.',null);
    insert into domains values ('Movies','Select AI','Natural language query for questions related to movies, actors, studios, pizzas, genre, or customers use this service.','OCI_LLAMA_INSTRUCT_NL2SQL');
    insert into domains values ('Schools','Select AI','Natural language query for questions related to high schools and what they offer.','OCI_LLAMA_INSTRUCT_SCHOOLS');
    commit;
    </copy>
    ```

9. Create Sequences

    ```
    <copy>
	create sequence conv_seq;
	create sequence prompt_seq;
    </copy>
    ```


10. Create Chat History Package and Package Body

    ```
    <copy>
    create or replace package chathistory_pkg is
    procedure prc_add_conversation(p_username varchar2, p_started_on timestamp, p_app_session number);
    procedure prc_rebuild_vectors;
    procedure prc_add_prompt_doc(
        p_doc_id number
        , p_conv_id number
        , p_prompt_id number
        , p_chunk_id number
        , p_index_id number
        , p_chunk varchar2);
    procedure prc_add_prompt(
        p_id number
        , p_conv_id number
        , p_prompt varchar2
        , p_response varchar2
        , p_asked_on timestamp
        , p_chathistory varchar2
        , p_references varchar2
        , p_request varchar2
        , p_output varchar2
        , p_citations varchar2
        , p_documents varchar2
        , p_documents4rerank varchar2
        , p_documentsreranked json
        , p_showsql varchar2
        , p_ai_service varchar2
        , p_domain varchar2);
    procedure prc_cr_doc_vectors(p_id number);
    procedure prc_cr_all_doc_vectors;
    end;
    /

    create or replace package body chathistory_pkg is
    ---------
    procedure prc_rebuild_vectors is
    begin
    update document_vector set embed_vector = null;
    commit;
    for c in 1..100 loop
    for i in (select * from document_vector where rownum<100 and embed_vector is null)
    loop
    update document_vector
    set embed_vector = dbms_vector.utl_to_embedding(chunk_txt, json('{
      "provider": "OCIGenAI",
      "credential_name": "{oci_vector_cred_from Task 4.3}",
      "url": "https://inference.generativeai.{region}.oci.oraclecloud.com/20231130/actions/embedText",
      "model": "cohere.embed-english-v3.0",
      "batch_size":10
    }'))
    where i.id = id
    and i.chunk_id = chunk_id;
    commit;
    end loop;
    end loop;
    end;
    --------
    procedure prc_add_conversation(p_username varchar2, p_started_on timestamp, p_app_session number) is
        pragma autonomous_transaction;
        begin
        insert into conversations values(conv_seq.nextval, p_username, p_started_on, p_app_session); 
        commit;
        end;
    ---------
    procedure prc_add_prompt_doc (
        p_doc_id number
        , p_conv_id number
        , p_prompt_id number
        , p_chunk_id number
        , p_index_id number
        , p_chunk varchar2) is
        pragma autonomous_transaction;
        begin
        insert into prompt_docs values(p_doc_id, p_conv_id, p_prompt_id, p_chunk_id, p_index_id, p_chunk);
        commit;
        end;
    ---------
    procedure prc_add_prompt(
        p_id number
        , p_conv_id number
        , p_prompt varchar2
        , p_response varchar2
        , p_asked_on timestamp
        , p_chathistory varchar2
        , p_references varchar2
        , p_request varchar2
        , p_output varchar2
        , p_citations varchar2
        , p_documents varchar2
        , p_documents4rerank varchar2
        , p_documentsreranked json
        , p_showsql varchar2
        , p_ai_service varchar2
        , p_domain varchar2) is
        pragma autonomous_transaction;
        begin
        insert into prompts (
            id
            , conv_id
            , prompt
            , response
            , asked_on
            , chathistory
            , references
            , request
            , output
            , citations
            , documents
            , documents4rerank
            , documentsreranked
            , showsql
            , ai_service
            , domain)
            values(
            p_id
            , p_conv_id
            , p_prompt
            , p_response
            , p_asked_on
            , p_chathistory
            , p_references
            , p_request
            , p_output
            , p_citations
            , p_documents
            , p_documents4rerank
            , p_documentsreranked
            , p_showsql
            , p_ai_service
            , p_domain);
        commit;
        end;
    ---------   
    procedure prc_cr_doc_vectors (p_id number) is
        v_vector_by             varchar2(100);
        v_vector_max            number;
        v_vector_overlap        number;
        v_vector_split          varchar2(100);
        v_vector_language       varchar2(100);
        v_vector_normalize      varchar2(100);
        --
        begin
        --
        select vector_by, vector_max, vector_overlap, vector_split, vector_language, vector_normalize
        into v_vector_by, v_vector_max, v_vector_overlap, v_vector_split, v_vector_language, v_vector_normalize
        from ai_configuration;
        --
        insert into document_vector
        select p_id
            , json_value(c.column_value, '$.chunk_id' returning number) as chunk_id
            , json_value(c.column_value, '$.chunk_offset' returning number) as chunk_pos
            , json_value(c.column_value, '$.chunk_length' returning number) as chunk_size
            , replace(json_value(c.column_value, '$.chunk_data'),chr(10),'') as chunk_txt
            , dbms_vector_chain.utl_to_embedding(json_value(c.column_value, '$.chunk_data'), json('{
                "provider": "OCIGenAI",
                "credential_name": "{oci_vector_cred_from Task 4.3}",
                "url": "https://inference.generativeai.{region}.oci.oraclecloud.com/20231130/actions/embedText",
                "model": "cohere.embed-english-v3.0",
                "batch_size":10
                }')) embed_vector 
        from 
        ------- doc to text query ---------
        (select id
            , regexp_replace(dbms_vector_chain.utl_to_text (l.file_content, json('{"plaintext":"true","charset":"utf8"}')),'\s+',' ') file_text
        from documents l where id = p_id) t,
        ------- chunking ---------
        dbms_vector_chain.utl_to_chunks(t.file_text,
        json('{ "by":"'||v_vector_by||'",
               "max":"'||v_vector_max||'",
               "overlap":"'||v_vector_overlap||'",
               "split":"'||v_vector_split||'",
               "language":"'||v_vector_language||'",
               "normalize":"'||v_vector_normalize||'" }')) c
        where t.id = p_id;
        commit;
     end;
    --------
    procedure prc_cr_all_doc_vectors is
        v_vector_by             varchar2(100);
        v_vector_max            number;
        v_vector_overlap        number;
        v_vector_split          varchar2(100);
        v_vector_language       varchar2(100);
        v_vector_normalize      varchar2(100);
        --
        begin
        --
        select vector_by, vector_max, vector_overlap, vector_split, vector_language, vector_normalize
        into v_vector_by, v_vector_max, v_vector_overlap, v_vector_split, v_vector_language, v_vector_normalize
        from ai_configuration;
        --
        for i in (select id from documents where id not in (select id from document_vector)) loop
        --    
        insert into document_vector
        select l.id
            , json_value(c.column_value, '$.chunk_id' returning number) as chunk_id
            , json_value(c.column_value, '$.chunk_offset' returning number) as chunk_pos
            , json_value(c.column_value, '$.chunk_length' returning number) as chunk_size
            , replace(json_value(c.column_value, '$.chunk_data'),chr(10),'') as chunk_txt
            , dbms_vector_chain.utl_to_embedding(json_value(c.column_value, '$.chunk_data'), json('{
                "provider": "OCIGenAI",
                "credential_name": "{oci_vector_cred_from Task 4.3}",
                "url": "https://inference.generativeai.{region}.oci.oraclecloud.com/20231130/actions/embedText",
                "model": "cohere.embed-english-v3.0",
                "batch_size":10
                }')) embed_vector 
        from 
        ------- base table ---------
        (select id from documents where id =i.id) l,
        ------- doc to text query ---------
        (select id
            , regexp_replace(dbms_vector_chain.utl_to_text (l.file_content, json('{"plaintext":"true","charset":"utf8"}')),'\s+',' ') file_text
        from documents l where id=l.id) t,
        ------- chunking ---------
        dbms_vector_chain.utl_to_chunks(t.file_text,
        json('{ "by":"'||v_vector_by||'",
               "max":"'||v_vector_max||'",
               "overlap":"'||v_vector_overlap||'",
               "split":"'||v_vector_split||'",
               "language":"'||v_vector_language||'",
               "normalize":"'||v_vector_normalize||'" }')) c
        where l.id=t.id;
        commit;
        end loop;
     end;
    end;
    /    
    </copy>
    ```

11. Create Moviestream Select AI Profile

    ```
    <copy>
    begin
        dbms_cloud_ai.create_profile(
            profile_name => 'oci_llama_instruct_nl2sql',
            attributes =>
                '{"provider": "oci",
                "credential_name": "{oci_api_cred_from Task 4.1}",
                "comments": true,
                "conversation": true,
                "model": "meta.llama-3.1-405b-instruct",
                "object_list": [{"owner": "search23ai", "name": "movies"},
                    {"owner": "moviestream", "name": "streams"},
                    {"owner": "moviestream", "name": "pizza_shop"},
                    {"owner": "moviestream", "name": "actors"},
                    {"owner": "moviestream", "name": "genre"},
                    {"owner": "moviestream", "name": "customer"}
                    ]
                }'
        );
    end;
    /
    </copy>
    ```

12. Create Schools Select AI Profile

    ```
    <copy>
    begin
        dbms_cloud_ai.create_profile(
            profile_name => 'oci_llama_instruct_schools',
            attributes =>
                '{"provider": "oci",
                "credential_name": "{oci_api_cred_from Task 4.1}",
                "comments": true,
                "conversation": true,
                "model": "meta.llama-3.1-405b-instruct",
                "object_list": [{"owner": "search23ai", "name": "highschools"}
                    ]
                }'
        );
    end;
    /
    </copy>
    ```

13. Create Document Vector Trigger

    ```
    <copy>
    -- kick off job to create vectors for new documents, updates document size, creates text from pdfs and populates clob column
    create or replace trigger trg_document_vector_ocigenai 
    before insert on documents
    for each row
    declare
    my_job number;
    begin
    dbms_job.submit(job => my_job, what => 'chathistory_pkg.prc_cr_doc_vectors('||:new.id||');');
    :new.file_size := dbms_lob.getlength(:new.file_content);
    -- remove CRLF and spaces when creating clob
    :new.file_content_clob := regexp_replace(dbms_vector_chain.utl_to_text (:new.file_content, json('{"plaintext":"true","charset":"utf8"}')),'\s+',' ');
    end;
    /
    </copy>
    ```

14. Create Vector CONVERSATION\_V View

    ```
    <copy>
    -- conversation_v - used in APEX app to view conversation prompts
    create or replace force editionable view conversation_v as 
      select
        c.id conversation_id
        , c.username
        , c.started_on
        , c.app_session
        , p.id prompt_id
        , p.prompt
        , p.response
        , p.asked_on
        , p.chathistory
        , p.references
        , p.request
        , p.output
        , p.citations
        , p.documents
        , p.documents4rerank
        , p.documentsreranked
        , p.showsql
        , p.ai_service
        , p.domain
    from conversations c, prompts p
    where p.conv_id = c.id;    
    </copy>
    ```

15. Create Vector DOCUMENT\_RANKING\_V View

    ```
    <copy>
    -- document_ranking_v - used in APEX to provide flattened JSON view of document ranking
    create or replace force editionable view document_ranking_v as
      select l.file_name
        , p.id prompt_id
        , p.conv_id
        , p.prompt
        , p.response
        , jt.documentid
        , jt.index_id
        , jt.score
        , jt.content
    from prompts p
        , json_table(p.output, '$.chatResponse.documents[*]' columns
            nested path '$[*]' columns (
                documentid varchar2(1000) path '$.id',
                index_id number path '$.index',
                score number path '$.score',
                content varchar2(1000) path '$.content'
                    )
            ) jt
        , prompt_docs d
        , documents l
    where p.conv_id = d.conv_id
    and p.id = d.prompt_id
    and jt.index_id = d.index_id
    and d.document_id = l.id;
    </copy>
    ```

16. Create Vector DOC\_CITATIONS\_V View

    ```
    <copy>
    -- doc_citations_v - used in APEX to provide flattened JSON view of citiations
    create or replace force editionable view doc_citations_v  as 
      select  d.file_name
        , p.id prompt_id
        , p.conv_id
        , p.asked_on
        , p.prompt
        , p.response
        , jt.citation_startpoint
        , jt.citation_endpoint
        , jt.citation_text
        , jt.documentids
    from prompts p, document_ranking_v d
        , json_table(p.output, '$.chatResponse.citations[*]' columns
            nested path '$[*]' columns (
                citation_startpoint number path '$.start',
                citation_endpoint number path '$.end',
                citation_text varchar2(100) path '$.text',
                nested path '$.documentIds[*]' columns (
                    documentIds varchar2(100) path '$')
                    )
            ) jt
    where p.conv_id = d.conv_id
    and p.id = d.prompt_id
    and jt.documentids = d.documentid;    
    </copy>
    ```

17. Create High Schools Table

    ```
    <copy>
    CREATE TABLE HIGHSCHOOLS 
        (ID_1                       NUMBER GENERATED BY DEFAULT ON NULL AS IDENTITY MINVALUE 1 MAXVALUE 9999999999999999999999999999 INCREMENT BY 1 START WITH 1 CACHE 20 NOORDER  NOCYCLE  NOKEEP  NOSCALE , 
        ID                          NUMBER, 
        BOROUGH                     VARCHAR2(50 BYTE), 
        SCHOOL_NAME                 VARCHAR2(255 BYTE), 
        NEIGHBORHOOD                VARCHAR2(50 BYTE), 
        INTEREST                    VARCHAR2(255 BYTE), 
        METHOD                      VARCHAR2(50 BYTE), 
        TOTAL_STUDENTS              NUMBER, 
        GRADUATION_RATE             NUMBER, 
        ATTENDANCE_RATE             NUMBER, 
        COLLEGE_CAREER_RATE         NUMBER, 
        SAFE                        NUMBER, 
        SEATS                       NUMBER, 
        APPLICANTS                  NUMBER, 
        DBN                         VARCHAR2(50 BYTE), 
        LATITUDE                    NUMBER, 
        LONGITUDE                   NUMBER, 
        LANGUAGE_CLASSES            VARCHAR2(255 BYTE), 
        ADVANCED_PLACEMENT_COURSES  VARCHAR2(32767 BYTE), 
        SCHOOL_SPORTS               VARCHAR2(32767 BYTE), 
        FAQ_URL                     VARCHAR2(32767 BYTE), 
        OVERVIEW_PARAGRAPH          VARCHAR2(32767 BYTE), 
        ACADEMIC_OPPORTUNITIES      VARCHAR2(32767 BYTE), 
        ADVANCEDPLACEMENT_COURSES   VARCHAR2(32767 BYTE), 
        DIADETAILS                  VARCHAR2(255 BYTE), 
        EXTRACURRICULAR_ACTIVITIES  VARCHAR2(32767 BYTE), 
        PSAL_SPORTS_BOYS            VARCHAR2(32767 BYTE), 
        PSAL_SPORTS_GIRLS           VARCHAR2(255 BYTE), 
        ADDTL_INFO1                 VARCHAR2(32767 BYTE)
        );
    </copy>
    ```

## Task 6: Load ONXX Models

As the user **SEARCH23AI**, issue the below SQL, PL/SQL Code.

More details on how-to leverage ONXX Models can be found below.

* [Import Pretrained Models in ONNX Format for Vector Generation Within the Database](https://docs.oracle.com/en/database/oracle/oracle-database/23/vecse/import-pretrained-models-onnx-format-vector-generation-database.html)

Oracle is providing a Hugging Face **all-MiniLM-L12-v2** model in ONNX format, available to download directly to the database using DBMS\_VECTOR.LOAD\_ONNX\_MODEL.  Please reference the below link to download the **all-MiniLM-L12-v2** model.

* [ONNX Pipeline Models : Text Embedding](https://docs.oracle.com/en/database/oracle/oracle-database/23/vecse/onnx-pipeline-models-text-embedding.html)


1. Copy ONXX Models from OCI Object Storage to Oracle Database Directory

    ```
    <copy>
	begin
	dbms_cloud.get_object(
		credential_name => '{update with oci_api_cred from Task 4.1}',
		object_uri => 'https://objectstorage.{region}.oraclecloud.com/n/{namespace}/b/{bucket_name}/o/all_MiniLM_L12_v2.onnx',
		directory_name => 'data_pump_dir'
		);
	end;
	/	
    </copy>
    ```

2. Load ONXX Models into Oracle Database

    ```
    <copy>
    exec DBMS_VECTOR.LOAD_ONNX_MODEL('data_pump_dir','all_MiniLM_L12_v2.onnx','All_MINILM_L12V2MODEL',JSON('{"function" : "embedding",'||'"input":{"input":["DATA"]}}'));
    </copy>
    ```

3. Chunks to Vectors (embeddings) using an ONXX Model 

    ```
    <copy>
	-- embedding query using ONXX model
	select chunk_txt, dbms_vector_chain.utl_to_embedding(l.chunk_txt, json('{
	  "provider": "database",
      "model": "All_MINILM_L12V2MODEL"
	}')) embed_vector
	from document_vector l
	where rownum < 11;	
    </copy>
    ```


## Task 7: Load Data into Database

As the user **SEARCH23AI**, issue the below SQL, PL/SQL Code.

1. Load Moviestream Data (takes approximatley 3 minutes)

    ```
    <copy>
    begin
        workshop.add_dataset 
            (tag => 'gen-ai',
            run_post_load_procedures => true
            );   
    end;
    /
    </copy>
    ```

2. Update Moviestream Views

    ```
    <copy>
    alter view v_target_customers compile;

    -- modify view to add primary key - to be used by global search index
    create or replace view movies
    (    movie_id
        , title
        , image_url
        , budget
        , gross
        , list_price
        , year
        , opening_date
        , genres
        , cast
        , crew
        , studio
        , main_subject
        , awards
        , nominations
        , runtime
        , summary
        , constraint movie_id_pk primary key (movie_id) rely disable novalidate)
    as select 
     json_value(json_document, '$.movie_id' returning number) as movie_id
     , json_value(json_document, '$.title' returning varchar2(200 byte)) as title
     , json_value(json_document, '$.image_url' returning varchar2(4000 byte)) as image_url
     , json_value(json_document, '$.budget' returning number) as budget
     , json_value(json_document, '$.gross' returning number) gross
     , json_value(json_document, '$.list_price' returning number) as list_price
     , json_value(json_document, '$.year' returning number) as year
     , to_date(json_value(json_document, '$.opening_date' returning varchar2(20)), 'yyyy-mm-dd') as opening_date
     , nullif(json_query(json_document, '$.genre' returning varchar2(4000 byte)), 'null') as genres
     , nullif(json_query(json_document, '$.cast' returning varchar2(4000 byte)), 'null') as cast
     , nullif(json_query(json_document, '$.crew' returning varchar2(4000 byte)), 'null') as crew
     , nullif(json_query(json_document, '$.studio' returning varchar2(4000 byte)), 'null') as studio
     , json_value(json_document, '$.main_subject' returning varchar2(4000 byte)) as main_subject
     , nullif(json_query(json_document, '$.awards' returning varchar2(4000 byte)),'null') as awards
     , nullif(json_query(json_document, '$.nominations' returning varchar2(4000 byte)),'null') as nominations
     , json_value(json_document, '$.runtime' returning number) as runtime
     , json_value(json_document, '$.summary' returning clob) as summary
      from "movieCollection" m;
    </copy>
    ```

3.  Load High Schools Data

    Using SQL Developer, SQL Developer Web or similar tool, execute the following SQL Script as user **search23ai**  
    
    * [High Schools Data ](https://github.com/snicholspa/tips_tricks_howtos/blob/main/autonomous_database/search23ai/files/insert_highschools_data.sql)

## Task 8: Load Documents into Database

As the user **SEARCH23AI**, issue the below SQL, PL/SQL Code.

Load your own PDFs into an Object Storage Bucket.  

To access the documents used in this example, you can download them from the following link

* [Sample PDFs](https://github.com/snicholspa/tips_tricks_howtos/blob/main/autonomous_database/search23ai/files/sample_pdfs.zip)

1. Load PDF Files from OCI Object Storage

    ```
    <copy>
    declare
       l_blob blob := null;
       l_bucket varchar2(4000) := 'https://objectstorage.{region}.oraclecloud.com/n/{namespace}/b/{bucket_name}/o/';
    begin
    for i in (select * from dbms_cloud.list_objects('{oci_api_cred_from Task 4.1}',l_bucket) where object_name like '%.pdf')
    loop
       l_blob := dbms_cloud.get_object(
         credential_name => '{oci_api_cred_from Task 4.1}',
         object_uri => l_bucket||i.object_name);
    insert into documents (file_name, file_size, file_type, file_content)
                values(i.object_name, i.bytes, 'application/pdf',l_blob);
    commit;
    end loop;
    end;
    /    
    </copy>
    ```

	**STOP and PLEASE Read the following step**

2.  At this point, you should have a populated **documents** and **document_vector** tables and the PDFs were chunked and vectorized (embeddings generated) and ready for searching.  To verify this, proceed to **Task 11**.

## Task 9: Create Database Objects - Functions and Hybrid Search Setup

As the user **SEARCH23AI**, issue the below SQL Code.

**Note** - Be sure to update the SQL Code accordingly where you see **{....}**

1. Add Contraint to High School Table 

    ```
    <copy>
    alter table highschools add primary key (id_1) enable;    
    </copy>
    ```

2. Vectorize High School Data (optional)

    ```
    <copy>
    alter table highschools add(embed_vector vector);

    update highschools set overview_paragraph='n/a' where overview_paragraph is null;
    commit;

    update highschools
    set embed_vector = dbms_vector.utl_to_embedding(overview_paragraph, json('{
      "provider": "OCIGenAI",
      "credential_name": "{oci_api_cred_from Task 4.3}",
      "url": "https://inference.generativeai.{region}.oci.oraclecloud.com/20231130/actions/embedText",
      "model": "cohere.embed-english-v3.0",
      "batch_size":10
    }'));

    commit;
    </copy>
    ```

3. Add Global Search Indexes

    ```
    <copy>
    exec dbms_search.create_index('global_idx');

    exec dbms_search.add_source('global_idx', 'search23ai.documents', '1g',4);
    exec dbms_search.add_source('global_idx', 'search23ai.highschools', '1g',4);
    exec dbms_search.add_source('global_idx', 'search23ai.movies', '1g',4);
    </copy>
    ```

4. Create NL2SQL Function

    ```
    <copy>
    create or replace function nl2sql (    
        p_ai_message in varchar2
        , p_domain in varchar2 default 'Default'
        , p_ai_profile in varchar2
        , p_session_id in number default sys_context('userenv','sessionid')
        ) return clob as
        v_response          varchar2(32767);
        v_response2          varchar2(4000);
        v_ai_service        varchar2(100);
        v_domain            varchar2(100);
        v_sql               varchar2(4000);
        v_domain            varchar2(100);
        v_session           number;
        v_conv_id           number;
        v_prompt_id         number;
        v_next_prompt_id    number;
    ---------------------------------------

    begin

    --------------------------
    -- Set conv_id and prompt_id
    --------------------------

    select count(*) into v_session from conversations where app_session = p_session_id; 
    if v_session = 0 then
    chathistory_pkg.prc_add_conversation(user, systimestamp, p_session_id); 
    end if;

    select prompt_seq.nextval into v_next_prompt_id;

    -- locate latest chat and prompt sessions and set current chat history
    select max(id) into v_conv_id from conversations where app_session = p_session_id;

    --
    begin
    select max(id) into v_prompt_id from prompts where conv_id = v_conv_id;
    exception
    when others then null;
    end;

    --------------------------------
    -- send request
    --------------------------------

    select dbms_cloud_ai.generate(
        prompt       => p_ai_message,
        profile_name => p_ai_profile,
        action       => 'narrate') into v_response;

    select dbms_cloud_ai.generate(
        prompt       => p_ai_message,
        profile_name => p_ai_profile,
        action       => 'showsql') into v_response2;

    --------------------------------
    -- save prompts
    --------------------------------
    chathistory_pkg.prc_add_prompt(
        v_next_prompt_id
        , v_conv_id
        , p_ai_message
        , v_response
        , systimestamp
        , null
        , null
        , null
        , null
        , null
        , null
        , null
        , null
        , v_response2
        , 'Select AI'
        , p_domain);
        --
    -- show me what you got
    return v_response;

    ---------------------------------------------
    end;
    /    
    </copy>
    ```

5. Create Gen AI Chat With Documents Function

    ```
    <copy>
    create or replace function gen_ai_chat_documents (
        p_ai_message in varchar2
        , p_domain in varchar2 default 'Default'
        , p_session_id in number
        ) return clob as
        ----
        v_gen_ai_endpoint   varchar2(500)   := 'https://inference.generativeai.{region}.oci.oraclecloud.com';
        v_compartment_id    varchar2(4000)  := 'ocid1.compartment.oc1..{change me}';
        v_vector_credential varchar2(100)   := '{oci_api_cred_from Task 4.3}';
        v_ociapi_credential varchar2(100)   := '{oci_api_cred_from Task 4.1}';
        v_provider          varchar2(100)   := 'OCIGenAI';
        v_text_endpoint     varchar2(100)   := '/20231130/actions/embedText';
        v_chat_endpoint     varchar2(100)   := '/20231130/actions/chat';
        v_model_embed       varchar2(100)   := 'cohere.embed-english-v3.0';
        v_model_query       varchar2(100)   := 'cohere.command-r-plus-08-2024';
        ----
        v_resp              dbms_cloud_types.resp;
        v_messages          varchar2(32767);
        v_output            varchar2(32767);
        v_ai_message_vec    vector;
        v_search_query      varchar2(32767);
        v_chunks            varchar2(32767);
        v_response          varchar2(32767);
        v_chathistory_after varchar2(32767);
        v_chathistory_before varchar2(32767);
        v_last_question     varchar2(32767) := ' ';
        v_top_doc_chunks    number;
        v_session           number;
        v_conv_id           number;
        v_prompt_id         number;
        v_next_prompt_id    number;
        v_prompt_instruction varchar2(32767);
        v_citations         varchar2(32767);
        v_documents         varchar2(32767);
        v_documents4rerank  varchar2(32767);
        v_documentsreranked json;
        v_doc varchar2(32767);
        v_cohere_params     varchar2(32767);
        v_index_id          number;
        ----
        v_ai_max_tokens     number;
        v_ai_temperature    number;
        v_ai_freqpenalty    number;
        v_ai_prespenalty    number;
        v_ai_topp           number;
        v_ai_topk           number;
        v_ai_vectorby       varchar2(100);
        v_ai_vectormax      number;
        v_ai_vectoroverlap  number;
        v_ai_split          varchar2(100);
        v_ai_language       varchar2(100);
        v_ai_normalize      varchar2(100);

    begin

    --------------------------
    -- Set conv_id and prompt_id
    --------------------------

    select count(*) into v_session from conversations where app_session = p_session_id; 
    if v_session = 0 then
    chathistory_pkg.prc_add_conversation(user, systimestamp, p_session_id); 
    end if;

    select prompt_seq.nextval into v_next_prompt_id;

    -- locate latest chat and prompt sessions and set current chat history
    select max(id) into v_conv_id from conversations where app_session = p_session_id;

    --
    begin
    select max(id) into v_prompt_id from prompts where conv_id = v_conv_id;
    exception
    when others then null;
    end;

    --------------------------------
    -- retrieve configuration
    --------------------------------

    select llm_max_tokens, llm_temperature, llm_frequencypenalty, llm_presencepenalty, llm_topp, llm_topk, top_doc_chunks
    into v_ai_max_tokens, v_ai_temperature, v_ai_freqpenalty, v_ai_prespenalty, v_ai_topp, v_ai_topk, v_top_doc_chunks
    from ai_configuration;

    --------------------------------
    -- Retrieve previous question and answer to prepend to the current question.  can go back further if need be
    --------------------------------

    begin
    select p.prompt||' '||p.response||'.  ' into v_last_question 
    from    prompts p
            , conversations c
    where   p.conv_id = c.id
    and     p.id = (select max(id) from prompts where p.conv_id = c.id)
    and     c.app_session = p_session_id
    and     p.ai_service = 'Vector Search'
    and     c.id = (select max(id) from conversations where p.conv_id = c.id);
    exception when others then null;
    end;
    -- dbms_output.put_line(v_last_question);

    --------------------------------
    -- vectorize user question
    --------------------------------

    select dbms_vector.utl_to_embedding(v_last_question||p_ai_message
        , json('{
            "provider":"'||v_provider||
            '","credential_name":"'||v_vector_credential||
            '","url":"'||v_gen_ai_endpoint||v_text_endpoint||
            '","model":"'||v_model_embed||'"}'))
    into v_ai_message_vec;

    v_documents := '{"documents":[';
    v_messages := '{"message": "';
    v_index_id := 0;

    -----------------------------------------------
    -- retrieve chunks based on vector distance 
    -- from input message and append to each other
    -----------------------------------------------
    for i in (select l.id, l.file_name, lv.chunk_id, replace(lv.chunk_txt,'"','') chunk_txt 
              from document_vector lv, documents l
              where l.id = lv.id  and rownum < 9999999999
              and l.domain = p_domain
              order by vector_distance(embed_vector, v_ai_message_vec, cosine) fetch first v_top_doc_chunks rows only)
    loop
        v_documents := v_documents||'"'||i.chunk_txt||'",';
        v_chunks := v_chunks||'Doc Id: '||to_char(i.id)||', Doc Name: '||to_char(i.file_name)||', chunk_id:'||to_char(i.chunk_id)||'; ';
        chathistory_pkg.prc_add_prompt_doc(i.id, v_conv_id, v_next_prompt_id, i.chunk_id, v_index_id, i.chunk_txt);
        v_index_id := v_index_id + 1;
    end loop;

    -- add the prompt instructions

    select instruction into v_prompt_instruction from prompt_instructions
    where active = 'Y';

    v_messages := '{"'||v_prompt_instruction||p_ai_message||'"}';

    -- remove the trailing comma and newline character

    v_messages := rtrim(v_messages, ',');
    v_chunks := rtrim(v_chunks,',');
    v_documents := rtrim(v_documents,',')||']}';
    -- dbms_output.put_line(v_documents);
    -- dbms_output.put_line(v_messages);
    --
    begin
    select chathistory into v_chathistory_before from prompts where id = v_prompt_id;
    exception
    when others then null;
    end;

    --------------------------------
    -- Rank the chunks for 
    --------------------------------

    v_cohere_params := '
    { "provider": "cohere",
      "credential_name": "COHERE_CRED",
      "url": "https://api.cohere.com/v2/rerank",
      "model": "rerank-english-v3.0",
      "return_documents": true,
      "top_n": '||v_top_doc_chunks||'}';

    -- add quotes around numeric values in json returned by dbms_vector.rerank  function
    select json_arrayagg(json_object("index" , "score" , "content" ))
    into v_documentsreranked
    from json_table (json(dbms_vector.rerank(p_ai_message, json(v_documents), json(v_cohere_params))), 
        '$[*]' columns("index", "score", "content" ));

    --------------------------------
    -- process response
    --------------------------------

    v_resp := dbms_cloud.send_request(
    credential_name => v_ociapi_credential,
    uri => v_gen_ai_endpoint || v_chat_endpoint,
    method => dbms_cloud.method_post,
    body => utl_raw.cast_to_raw(json_object(
        'compartmentId'     value v_compartment_id,
        'servingMode'       value
            (json_object(
                'modelId'               value v_model_query,
                'servingType'           value 'ON_DEMAND'
                )),
                'chatRequest'           value 
                (json_object(           
                    'message'           value v_messages,
                    'chatHistory'       value json(v_chathistory_before),
                    'apiFormat'         value 'COHERE',
                    'maxTokens'         value v_ai_max_tokens,
                    'temperature'       value v_ai_temperature,
                    'frequencyPenalty'  value v_ai_freqpenalty,
                    'presencePenalty'   value v_ai_prespenalty,
                    'topP'              value v_ai_topp,
                    'topK'              value v_ai_topk,
                    'documents'         value v_documentsreranked,
                    ---- defaults
                    'citationQuality'   value 'ACCURATE',
                    'isEcho'            value false,
                    'isForceSingleStep' value false,
                    'isRawPrompting'    value false,
                    'isSearchQueriesOnly' value false,
                    'isStream'          value false,
                    'promptTruncation'  value 'OFF'
                    /* ---- others
                    maxinputTokens
                    preambleOverride
                    seed
                    stopSequences
                    tools
                    toolResults 
                    https://docs.oracle.com/en-us/iaas/api/#/en/generative-ai-inference/20231130/datatypes/BaseChatRequest
                    */
                    ))
                ))
            );

    v_output            := dbms_cloud.get_response_text(v_resp);
    v_response          := json_value(v_output, '$.chatResponse.text' returning varchar2);
    v_chathistory_after := json_query(v_output, '$.chatResponse.chatHistory' returning varchar2);
    v_citations         := json_query(v_output, '$.chatResponse.citations' returning varchar2);
    v_documents         := json_query(v_output, '$.chatResponse.documents' returning varchar2);

    --------------------------------
    -- save prompts
    --------------------------------
    chathistory_pkg.prc_add_prompt(
        v_next_prompt_id
        , v_conv_id
        , p_ai_message
        , v_response
        , systimestamp
        , v_chathistory_after
        , v_chunks
        , v_messages
        , v_output
        , v_citations
        , v_documents
        , v_documents4rerank
        , v_documentsreranked
        , null
        , 'Vector Search'
        , p_domain);

    -- show me what you got
    return v_response;

    end;
    /
    </copy>
    ```

6. Create AI Search Function

    ```
    <copy>
    create or replace function ai_search (    
        p_ai_message in varchar2
        , p_domain in varchar2 default 'Default'
        , p_session_id in number default sys_context('userenv','sessionid')
        ) return clob as
        v_response          varchar2(32767);
        v_ai_service        varchar2(100);
        v_domain            varchar2(100);
        v_ai_profile        varchar2(100);
    ---------------------------------------
    begin

    begin
    select ai_service, ai_profile into v_ai_service, v_ai_profile from domains where domain_name = p_domain;
    exception when others then null;
    end;

    if v_ai_service = 'Vector Search' then
        select gen_ai_chat_documents(p_ai_message, p_domain, p_session_id) into v_response;
    else select nl2sql(p_ai_message, p_domain, v_ai_profile, p_session_id) into v_response;
    end if;

    return v_response;
    ---------------------------------------------
    end;
    /    
    </copy>
    ```

7. Create Gen AI Chat With Documents Single Function

    ```
    <copy>
    create or replace function gen_ai_chat_documents_single (
        p_ai_message in varchar2
        , p_file_name in varchar2
        , p_session_id in number default sys_context('userenv','sessionid')
        ) return clob as
        ----
        v_gen_ai_endpoint   varchar2(500)   := 'https://inference.generativeai.{region}.oci.oraclecloud.com';
        v_compartment_id    varchar2(4000)  := 'ocid1.compartment.oc1..{change me}';
        v_vector_credential varchar2(100)   := '{oci_api_cred_from Task 4.3}';
        v_ociapi_credential varchar2(100)   := '{oci_api_cred_from Task 4.1}';
        v_provider          varchar2(100)   := 'OCIGenAI';
        v_text_endpoint     varchar2(100)   := '/20231130/actions/embedText';
        v_chat_endpoint     varchar2(100)   := '/20231130/actions/chat';
        v_model_query       varchar2(100)   := 'meta.llama-3.2-90b-vision-instruct';
        ----
        v_resp              dbms_cloud_types.resp;
        v_messages          clob;
        v_output            clob;
        v_response          clob;
        v_session           number;
        v_conv_id           number;
        v_prompt_id         number;
        v_next_prompt_id    number;
        v_domain            varchar2(100);
        ----
        v_ai_prompt_document_contents clob;
        v_prompt_instruction varchar2(32767);
        v_request_body      blob;
        v_request_json_part1 clob;
        v_request_json_part2 clob;
        ----
        v_ai_max_tokens     number;
        v_ai_temperature    number;
        v_ai_freqpenalty    number;
        v_ai_prespenalty    number;
        v_ai_topp           number;
        v_ai_topk           number;

    begin

    --------------------------
    -- retrieve document
    --------------------------

    dbms_lob.createtemporary(v_request_body, false);

    select p_ai_message || ': ' || REGEXP_REPLACE(file_content_clob, '[^a-zA-Z ,.]') 
    into v_ai_prompt_document_contents 
    from documents where file_name = p_file_name;

    --------------------------
    -- Set conv_id and prompt_id
    --------------------------

    select count(*) into v_session from conversations where app_session = p_session_id; 
    if v_session = 0 then
    chathistory_pkg.prc_add_conversation(user, systimestamp, p_session_id); 
    end if;

    select prompt_seq.nextval into v_next_prompt_id;

    -- locate latest chat and prompt sessions and set current chat history
    select max(id) into v_conv_id from conversations where app_session = p_session_id;

    --
    begin
    select max(id) into v_prompt_id from prompts where conv_id = v_conv_id;
    exception
    when others then null;
    end;

    --------------------------------
    -- retrieve configuration
    --------------------------------

    select llm_max_tokens, llm_temperature, llm_frequencypenalty, llm_presencepenalty, llm_topp, llm_topk
    into v_ai_max_tokens, v_ai_temperature, v_ai_freqpenalty, v_ai_prespenalty, v_ai_topp, v_ai_topk
    from ai_configuration;

    -- add the prompt instructions
    select instruction into v_prompt_instruction from prompt_instructions
    where active = 'Y';

    --------------------------------
    -- retrieve domain
    --------------------------------

    select domain into v_domain from documents where file_name = p_file_name;

    --------------------------------
    -- build request
    --------------------------------
    v_request_json_part1 := to_clob(
         '{
            "compartmentId": "ocid1.compartment.oc1..{change me}",
            "servingMode": 
                {
                    "modelId": "meta.llama-3.2-90b-vision-instruct",
                    "servingType": "ON_DEMAND"
                }
            ,
            "chatRequest": {
                "apiFormat": "GENERIC",	
                "messages": [
                    {
                        "role": "USER",				
                        "content": [
                            {
                                "type": "TEXT",
                                "text": "');

    v_request_json_part2 := to_clob('"
                            }
                        ]
                    }
                ],
                "maxTokens":         2000,
                "temperature":       0.75,
                "numGenerations":    1,
                "topK":              -1,
                "isStream":          false			
            }
        }');

    -- append part1 json to request blob
    dbms_lob.append(v_request_body, apex_util.clob_to_blob(p_clob => v_request_json_part1,p_charset => 'AL32UTF8'));

    -- append base64 image to request blob
    dbms_lob.append(v_request_body, apex_util.clob_to_blob(p_clob => v_ai_prompt_document_contents,p_charset => 'AL32UTF8'));

    -- append part2 json to request blob
    dbms_lob.append(v_request_body, apex_util.clob_to_blob(p_clob => v_request_json_part2,p_charset => 'AL32UTF8'));

    v_resp := dbms_cloud.send_request(
    credential_name => v_ociapi_credential,
    uri => v_gen_ai_endpoint || v_chat_endpoint,
    method => dbms_cloud.method_post,
    body => v_request_body);

    v_output            := dbms_cloud.get_response_text(v_resp);
    v_response          := json_value(v_output, '$.chatResponse.choices.message.content.text' returning clob);

    chathistory_pkg.prc_add_prompt(
        v_next_prompt_id
        , v_conv_id
        , p_ai_message
        , v_response
        , systimestamp
        , null
        , null
        , null
        , v_output
        , null
        , p_file_name
        , null
        , null
        , null
        , 'Document Search'
        , v_domain);

    -- show me what you got
    return v_response;

    end;
    /
    </copy>
    ```

8. Create Gen AI Chat With Documents Search Function

    ```
    <copy>
    create or replace function gen_ai_chat_documents_search (
        p_ai_message in varchar2
        , p_domain in varchar2 default 'Default'
        , p_contains in varchar2
        , p_session_id in number default sys_context('userenv','sessionid')
        ) return clob as
        ----
        v_gen_ai_endpoint   varchar2(500)   := 'https://inference.generativeai.{region}.oci.oraclecloud.com';
        v_compartment_id    varchar2(4000)  := 'ocid1.compartment.oc1..{change me}';
        v_vector_credential varchar2(100)   := '{oci_api_cred_from Task 4.3}';
        v_ociapi_credential varchar2(100)   := '{oci_api_cred_from Task 4.1}';
        v_provider          varchar2(100)   := 'OCIGenAI';
        v_text_endpoint     varchar2(100)   := '/20231130/actions/embedText';
        v_chat_endpoint     varchar2(100)   := '/20231130/actions/chat';
        v_model_embed       varchar2(100)   := 'cohere.embed-english-v3.0';
        v_model_query       varchar2(100)   := 'cohere.command-r-plus-08-2024';
        ----
        v_resp              dbms_cloud_types.resp;
        v_messages          varchar2(32767);
        v_output            varchar2(32767);
        v_ai_message_vec    vector;
        v_search_query      varchar2(32767);
        v_chunks            varchar2(32767);
        v_response          clob;
        v_chathistory_after varchar2(32767);
        v_chathistory_before varchar2(32767);
        v_last_question     varchar2(32767) := ' ';
        v_top_doc_chunks    number;
        v_session           number;
        v_conv_id           number;
        v_prompt_id         number;
        v_next_prompt_id    number;
        v_prompt_instruction varchar2(32767);
        v_citations         clob;
        v_documents         clob;
        v_documents4rerank  clob;
        v_documentsreranked json;
        v_doc varchar2(32767);
        v_cohere_params     varchar2(32767);
        v_index_id          number;
        ----
        v_ai_max_tokens     number;
        v_ai_temperature    number;
        v_ai_freqpenalty    number;
        v_ai_prespenalty    number;
        v_ai_topp           number;
        v_ai_topk           number;
        v_ai_vectorby       varchar2(100);
        v_ai_vectormax      number;
        v_ai_vectoroverlap  number;
        v_ai_split          varchar2(100);
        v_ai_language       varchar2(100);
        v_ai_normalize      varchar2(100);

    begin

    --------------------------
    -- Set conv_id and prompt_id
    --------------------------
    select count(*) into v_session from conversations where app_session = p_session_id; 
    if v_session = 0 then
    chathistory_pkg.prc_add_conversation(user, systimestamp, p_session_id); 
    end if;

    select prompt_seq.nextval into v_next_prompt_id;

    -- locate latest chat and prompt sessions and set current chat history
    select max(id) into v_conv_id from conversations where app_session = p_session_id;

    --
    begin
    select max(id) into v_prompt_id from prompts where conv_id = v_conv_id;
    exception
    when others then null;
    end;

    --------------------------------
    -- retrieve configuration
    --------------------------------
    select llm_max_tokens, llm_temperature, llm_frequencypenalty, llm_presencepenalty, llm_topp, llm_topk, top_doc_chunks
    into v_ai_max_tokens, v_ai_temperature, v_ai_freqpenalty, v_ai_prespenalty, v_ai_topp, v_ai_topk, v_top_doc_chunks
    from ai_configuration;

    --------------------------------
    -- vectorize user question
    --------------------------------
    select dbms_vector.utl_to_embedding(v_last_question||p_ai_message
        , json('{
            "provider":"'||v_provider||
            '","credential_name":"'||v_vector_credential||
            '","url":"'||v_gen_ai_endpoint||v_text_endpoint||
            '","model":"'||v_model_embed||'"}'))
    into v_ai_message_vec;

    v_documents := '{"documents":[';
    v_messages := '{"message": "';
    v_index_id := 0;

    -----------------------------------------------
    -- retrieve chunks based on vector distance 
    -- from input message and append to each other
    -----------------------------------------------
    for i in (select l.id, l.file_name, lv.chunk_id, replace(lv.chunk_txt,'"','') chunk_txt 
              from document_vector lv, documents l
              where l.id = lv.id  and rownum < 9999999999
              and l.domain = p_domain
              and l.id in (
                select json_value(key,'$.ID' returning number) key from global_idx
                where contains (data, p_contains) > 0 and source = 'DOCUMENTS') 
              order by vector_distance(embed_vector, v_ai_message_vec, cosine) fetch first v_top_doc_chunks rows only)
    loop
        v_documents := v_documents||'"'||i.chunk_txt||'",';
        v_chunks := v_chunks||'Doc Id: '||to_char(i.id)||', Doc Name: '||to_char(i.file_name)||', chunk_id:'||to_char(i.chunk_id)||'; ';
        chathistory_pkg.prc_add_prompt_doc(i.id, v_conv_id, v_next_prompt_id, i.chunk_id, v_index_id, i.chunk_txt);
        v_index_id := v_index_id + 1;
    end loop;

    -- add the prompt instructions
    select instruction into v_prompt_instruction from prompt_instructions
    where active = 'Y';

    v_messages := '{"'||v_prompt_instruction||p_ai_message||'"}';

    -- remove the trailing comma and newline character
    v_messages := rtrim(v_messages, ',');
    v_chunks := rtrim(v_chunks,',');
    v_documents := rtrim(v_documents,',')||']}';
    --
    begin
    select chathistory into v_chathistory_before from prompts where id = v_prompt_id;
    exception
    when others then null;
    end;

    --------------------------------
    -- Rank the chunks for 
    --------------------------------
    v_cohere_params := '
    { "provider": "cohere",
      "credential_name": "COHERE_CRED",
      "url": "https://api.cohere.com/v2/rerank",
      "model": "rerank-english-v3.0",
      "return_documents": true,
      "top_n": '||v_top_doc_chunks||'}';

    -- add quotes around numeric values in json returned by dbms_vector.rerank  function
    select json_arrayagg(json_object("index" , "score" , "content" ))
    into v_documentsreranked
    from json_table (json(dbms_vector.rerank(p_ai_message, json(v_documents), json(v_cohere_params))), 
        '$[*]' columns("index", "score", "content" ));

    --------------------------------
    -- process response
    --------------------------------
    v_resp := dbms_cloud.send_request(
    credential_name => v_ociapi_credential,
    uri => v_gen_ai_endpoint || v_chat_endpoint,
    method => dbms_cloud.method_post,
    body => utl_raw.cast_to_raw(json_object(
        'compartmentId'     value v_compartment_id,
        'servingMode'       value
            (json_object(
                'modelId'               value v_model_query,
                'servingType'           value 'ON_DEMAND'
                )),
                'chatRequest'           value 
                (json_object(           
                    'message'           value v_messages,
                    'chatHistory'       value json(v_chathistory_before),
                    'apiFormat'         value 'COHERE',
                    'maxTokens'         value v_ai_max_tokens,
                    'temperature'       value v_ai_temperature,
                    'frequencyPenalty'  value v_ai_freqpenalty,
                    'presencePenalty'   value v_ai_prespenalty,
                    'topP'              value v_ai_topp,
                    'topK'              value v_ai_topk,
                    'documents'         value v_documentsreranked,
                    ---- defaults
                    'citationQuality'   value 'ACCURATE',
                    'isEcho'            value false,
                    'isForceSingleStep' value false,
                    'isRawPrompting'    value false,
                    'isSearchQueriesOnly' value false,
                    'isStream'          value false,
                    'promptTruncation'  value 'OFF'
                    ))
                ))
            );

    v_output            := dbms_cloud.get_response_text(v_resp);
    v_response          := json_value(v_output, '$.chatResponse.text' returning varchar2);
    v_chathistory_after := json_query(v_output, '$.chatResponse.chatHistory' returning varchar2);
    v_citations         := json_query(v_output, '$.chatResponse.citations' returning varchar2);
    v_documents         := json_query(v_output, '$.chatResponse.documents' returning varchar2);

    --------------------------------
    -- save prompts
    --------------------------------
    chathistory_pkg.prc_add_prompt(
        v_next_prompt_id
        , v_conv_id
        , p_ai_message
        , v_response
        , systimestamp
        , v_chathistory_after
        , v_chunks
        , v_messages
        , v_output
        , v_citations
        , v_documents
        , v_documents4rerank
        , v_documentsreranked
        , null
        , 'Vector Search With Keyword'
        , p_domain);

    -- show me what you got
    return v_response;

    end;
    /
    </copy>
    ```

9. Create Chat with Image Function 

    ```
    <copy>
    create or replace FUNCTION gen_ai_chat_documents_image (
        p_ai_prompt IN VARCHAR2
        , p_image_name in VARCHAR2
        , p_session_id in number default sys_context('userenv','sessionid')
        ) RETURN CLOB AS
        ----
        v_gen_ai_endpoint       varchar2(500) := 'https://inference.generativeai.{region}.oci.oraclecloud.com';
        v_gen_ai_model          varchar2(500) := 'meta.llama-3.2-90b-vision-instruct';
        v_compartment_ocid      varchar2(500) := 'ocid1.compartment.oc1..{change me}';
        v_resp                  dbms_cloud_types.RESP;
        v_base64_image          CLOB;
        v_request_json_part1    CLOB;
        v_request_json_part2    CLOB;
        v_request_body          BLOB;
        v_blob_image            BLOB;
        v_output                clob;
        v_response              clob;
        v_session           number;
        v_conv_id           number;
        v_prompt_id         number;
        v_next_prompt_id    number;
        v_domain            varchar2(100);
    ----
    BEGIN
    --------------------------
    -- Set conv_id and prompt_id
    --------------------------
    select count(*) into v_session from conversations where app_session = p_session_id;
    if v_session = 0 then
    chathistory_pkg.prc_add_conversation(user, systimestamp, p_session_id);
    end if;
    select prompt_seq.nextval into v_next_prompt_id;
    -- locate latest chat and prompt sessions and set current chat history
    select max(id) into v_conv_id from conversations where app_session = p_session_id;
    --
    begin
    select max(id) into v_prompt_id from prompts where conv_id = v_conv_id;
    exception
    when others then null;
    end;
    --------------------------------
    -- retrieve domain
    --------------------------------
    select domain into v_domain from documents where file_name = p_image_name;
    --------------------------------
    -- create temp blobs
    --------------------------------
    dbms_lob.createtemporary(v_request_body, FALSE);
    --------------------------------
    -- get image blob from table
    --------------------------------
    select file_content into v_blob_image from documents where file_name = p_image_name;
    v_base64_image := APEX_WEB_SERVICE.BLOB2CLOBBASE64(v_blob_image,'N','N');
    --------------------------------
    -- build request
    --------------------------------
        v_request_json_part1 := to_clob(
         '{
            "compartmentId": "'||v_compartment_ocid||'",
            "servingMode":
                {
                    "modelId": "'||v_gen_ai_model||'",
                    "servingType": "ON_DEMAND"
                }
            ,
            "chatRequest": {
                "apiFormat": "GENERIC",	
                "messages": [
                    {
                        "role": "USER",				
                        "content": [
                            {
                                "type": "TEXT",
                                "text": "'||p_ai_prompt||'"
                            },
                            {
                                "type": "IMAGE",
                                "imageUrl": {
                                    "url": "data:image/png;base64,');
        v_request_json_part2 := to_clob('"
                                }
                            }
                        ]
                    }
                ],
                "maxTokens":         4000,
                "temperature":       0.75,
                "numGenerations":    1,
                "topK":              -1,
                "isStream":          false			
            }
        }');
    -- update request body for model and prompt
    v_request_json_part1 := replace(v_request_json_part1,v_gen_ai_model,v_gen_ai_model);
    v_request_json_part1 := replace(v_request_json_part1,p_ai_prompt,p_ai_prompt);
    -- append part1 json to request blob
    dbms_lob.append(v_request_body, apex_util.clob_to_blob(p_clob => v_request_json_part1, p_charset => 'AL32UTF8'));
    -- append base64 image to request blob
    dbms_lob.append(v_request_body, apex_util.clob_to_blob(p_clob => v_base64_image, p_charset => 'AL32UTF8'));
    -- append part2 json to request blob
    dbms_lob.append(v_request_body, apex_util.clob_to_blob(p_clob => v_request_json_part2, p_charset => 'AL32UTF8'));
    -- call Gen AI Chat
    v_resp := dbms_cloud.send_request(
        credential_name => 'oci_key_cred',
        uri => v_gen_ai_endpoint || '/20231130/actions/chat',
        method => dbms_cloud.METHOD_POST,
        body => v_request_body
    );
    v_output            := dbms_cloud.get_response_text(v_resp);
    v_response          := json_value(v_output, '$.chatResponse.choices.message.content.text' returning clob);
    -- free temp blobs
    dbms_lob.freetemporary(v_request_body);
    -- save prompt
    chathistory_pkg.prc_add_prompt(
        v_next_prompt_id
        , v_conv_id
        , p_ai_prompt
        , v_response
        , systimestamp
        , null
        , null
        , null
        , v_output
        , null
        , p_image_name
        , null
        , null
        , null
        , 'Image Search'
        , v_domain);
    -- show me what you got
    return v_response;
    END gen_ai_chat_documents_image;
    </copy>
    ```

10. Create Document Search Function

    ```
    <copy>
    create or replace function document_search (
          p_ai_prompt  in varchar2
        , p_file_name  in varchar2 default 'Default'
        , p_session_id in number default sys_context('userenv','sessionid')
        ) return clob as
        v_response         varchar2(32767);
        v_file_type        varchar2(100);
    ---------------------------------------
    begin
        begin
            select file_type into v_file_type from documents where file_name = p_file_name;
        exception when others then null;
        end;
        if v_file_type = 'application/pdf' then
            select gen_ai_chat_documents_single(p_ai_prompt, p_file_name, p_session_id) into v_response;
        else
            select gen_ai_chat_documents_image(p_ai_prompt, p_file_name, p_session_id) into v_response;
        end if;
        return v_response;
        ---------------------------------------------
    end;
    </copy>
    ```

## Task 10: Create Hybrid Search Index

As the user **SEARCH23AI**, issue the below SQL, PL/SQL Code.

1. Create **documents\_hybrid\_search** table

    ```
    <copy>
    create table documents_hybrid_search as select * from documents;
    alter table documents_hybrid_search add constraint documents_hybrid_search_pk primary key(id);
    </copy>
    ```
    
2. Create Vector Preference 

    ```
    <copy>
    begin
      DBMS_VECTOR_CHAIN.CREATE_PREFERENCE(
        'my_vectorizer_pref',
         dbms_vector_chain.vectorizer,
        json('{
                "vector_idxtype":  "hnsw",
                "model"         :  "ALL_MINILM_L12V2MODEL",
                "by"            :  "words",
                "max"           :  100,
                "overlap"       :  0,
                "split"         :  "sentence"
              }'
            ));
    end;
    /    
    </copy>
    ```

3. Create Hybrid Vector Index

    **Note:**  This steps takes approximately **75 - 90** minutes to complete

    ```
    <copy>
    create hybrid vector index documents_hybrid_search_idx on 
      documents_hybrid_search(file_content)
        parameters('vectorizer my_vectorizer_pref');    
    </copy>
    ```

## Task 11: Vector Text, Chunking, Embedding Manual Steps and Validation (Informational)

**STOP and PLEASE Read the following**

Performing Tasks 1-10 should have loaded the following

* PDF files stored in OCI Object Storage into the **documents** table 
* Loaded the Moviestream data in serverl tables
* Loaded the High Schools data into the **HIGHSCHOOLS** table
* Created and built the Hybrid Indexes

In addition, there's a **trigger (Task 5, Step 13)** on the **documents** table which will perform the following vectorization steps

* Populates the **documents.file\_content\_clob** column with the text from the PDF stored in the **documents.file\_content** column
* Performs the Vector chunking based on the settings in the **ai\_configuration** table and populates the **document\_vector** table
* Performs the Vector embeddings on the Vector chunks and populates the **document\_vector.embed\_vector** column.

To validate all the above steps were execute, you can execute the first SQL statement to list the chunk count by file name.  You should have a row for each file you loaded from Object Storage.  The second SQL statement will identify any chunks without a corresponding calculated Vector Embedding.  If you see chunks and or embeddings missing, please refer to the below steps to create the chunks and embeddings manually.

This counts the chunks by file.

```
<copy>
-- list chunk count by file name
select d.file_name, count(*)
from documents d, document_vector dv
where d.id = dv.id
group by d.file_name
order by d.file_name;
</copy>
```

This returns documents without vector embeddings generated.  It should return NO rows.

```
<copy>
-- list chunks without vectors count by file name
select d.file_name, count(*)
from documents d, document_vector dv
where d.id = dv.id and dv.embed_vector is null
group by d.file_name
order by d.file_name;
</copy>
```

This performs a Hybrid Search.

```
<copy>
select h.id
    , h.file_name
    , v.vector_score
    , v.text_score
from documents_hybrid_search h
    , (select d.id
        , d.vector_score
        , d.text_score
        from
        (select dbms_hybrid_vector.search(
    json(
      '{
         "hybrid_index_name" : "documents_hybrid_search_idx",
         "search_scorer"     : "rsf",
         "search_fusion"     : "UNION",
         "vector":
          {
             "search_text"   : "what legal documents address public safety",
             "search_mode"   : "DOCUMENT",
             "aggregator"    : "MAX",
             "score_weight"  : 1,
             "rank_penalty"  : 5
          },
         "text":
          {
             "contains"      : "invasion",
             "score_weight"  : 10,
             "rank_penalty"  : 1
          },
         "return":
          {
             "values"        : [ "rowid", "score", "vector_score", "text_score" ],
             "topN"          : 3
                }
            }'
            )) data) j,
        json_table ( j.data, '$[*]'
            columns (
                id varchar2(100) path '$."rowid"',
                vector_score number path '$."vector_score"',
                text_score number path '$."text_score"'
                        )
                    )
                d) v
where h.rowid = v.id
order by v.text_score desc, v.vector_score desc;
</copy>
```

Below are informational and demonstrates how to convert to text, chunk and vectorize a document.

As the user **SEARCH23AI**, issue the below SQL, PL/SQL Code.

1. Document to text

    ```
    <copy>
	-- just extract text
	select id, dbms_vector_chain.utl_to_text (l.file_content, json('{"plaintext":"true","charset":"utf8"}')) file_text
	from documents l;
    </copy>
    ```

2. Document to Text to Chunks

    ```
    <copy>
	-- chunking from text
	select l.id
			, l.file_name
			, l.file_size
			, json_value(c.column_value, '$.chunk_id' returning number) as chunk_id
			, json_value(c.column_value, '$.chunk_offset' returning number) as chunk_pos
			, json_value(c.column_value, '$.chunk_length' returning number) as chunk_size
			, replace(json_value(c.column_value, '$.chunk_data'),chr(10),'') as chunk_txt
	from 
	-- base table
	documents l,
	-- doc to text
	(select id
			, dbms_vector_chain.utl_to_text (l.file_content, json('{"plaintext":"true","charset":"utf8"}')) file_text
		from documents l) t,
	-- text to chunck
	dbms_vector_chain.utl_to_chunks(t.file_text,
	   json('{ "by":"words",
			   "max":"50",
			   "overlap":"0",
			   "split":"sentence",
			   "language":"american",
			   "normalize":"all"}')) c
	-- joins
	where l.id = t.id;
    </copy>
    ```

    ```
    <copy>
	insert into document_vector
	select l.id
			, json_value(c.column_value, '$.chunk_id' returning number) as chunk_id
			, json_value(c.column_value, '$.chunk_offset' returning number) as chunk_pos
			, json_value(c.column_value, '$.chunk_length' returning number) as chunk_size
			, replace(json_value(c.column_value, '$.chunk_data'),chr(10),'') as chunk_txt
			, null -- this is the vector column, we'll populate it later
	from 
	------- base table ---------
	documents l,
	------- doc to text query ---------
	(select id
			, dbms_vector_chain.utl_to_text (l.file_content, json('{"plaintext":"true","charset":"utf8"}')) file_text
		from documents l) t,
	------- chunking ---------
	dbms_vector_chain.utl_to_chunks(t.file_text,
	   json('{ "by":"words",
			   "max":"50",
			   "overlap":"0",
			   "split":"sentence",
			   "language":"american",
			   "normalize":"all" }')) c
	------- joins ---------
	where l.id = t.id;
	commit;
	</copy>
    ```

3. Chunks to Vectors (embeddings)

	This example leverages the **OCIGenAI** provider and the **Cohere** embedding model

    ```
    <copy>
	-- embedding query
	select chunk_txt, dbms_vector_chain.utl_to_embedding(l.chunk_txt, json('{
	  "provider": "OCIGenAI",
	  "credential_name": "{update with oci_vector_cred from Task 3.3}",
	  "url": "https://inference.generativeai.{region}.oci.oraclecloud.com/20231130/actions/embedText",
	  "model": "cohere.embed-english-v3.0",
	  "batch_size":10
	}')) embed_vector
	from document_vector l
	where rownum < 11;
    </copy>
    ```

	This example leverages the **database** provider and the ONXX **All\_MINILM\_L6V2MODEL** embedding model loaded in the **Task 6**

    ```
    <copy>
	-- embedding query using ONXX model
	select chunk_txt, dbms_vector_chain.utl_to_embedding(l.chunk_txt, json('{
	  "provider": "database",
      "model": "All_MINILM_L6V2MODEL"
	}')) embed_vector
	from document_vector l
	where rownum < 11;
    </copy>
    ```

    ```
    <copy>
	-- update vectors in micro batches, looping through rows that don't have vectors
	begin
	for c in 1..100 loop
	for i in (select * from document_vector where rownum<100 and embed_vector is null)
	loop
	update document_vector
	set embed_vector = dbms_vector.utl_to_embedding(chunk_txt, json('{
	  "provider": "OCIGenAI",
	  "credential_name": "{update with oci_vector_cred from Task 3.3}",
	  "url": "https://inference.generativeai.{region}.oci.oraclecloud.com/20231130/actions/embedText",
	  "model": "cohere.embed-english-v3.0",
	  "batch_size":10
	}'))
	where i.id = id
	and i.chunk_id = chunk_id;
	commit;
	end loop;
	end loop;
	end;
	/
    </copy>
    ```

4. Document to Text to Chunks to Vectors in One Flow (** MAY Fails base on Service Limits **)

    ```
    <copy>
	-- combine all the steps in one flow
	select l.id
			, l.file_name
			, l.file_size
			, json_value(c.column_value, '$.chunk_id' returning number) as chunk_id
			, json_value(c.column_value, '$.chunk_offset' returning number) as chunk_pos
			, json_value(c.column_value, '$.chunk_length' returning number) as chunk_size
			, replace(json_value(c.column_value, '$.chunk_data'),chr(10),'') as chunk_txt
			, dbms_vector_chain.utl_to_embedding(json_value(c.column_value, '$.chunk_data'), json('{
				"provider": "OCIGenAI",
				"credential_name": "{update with oci_vector_cred from Task 3.3}",
				"url": "https://inference.generativeai.{region}.oci.oraclecloud.com/20231130/actions/embedText",
				"model": "cohere.embed-english-v3.0",
				"batch_size":50
				}')) embed_vector
	from 
	-- base table
	documents l,
	-- doc to text
	(select id
			, dbms_vector_chain.utl_to_text (l.file_content, json('{"plaintext":"true","charset":"utf8"}')) file_text
		from documents l) t,
	-- text to chunks
	dbms_vector_chain.utl_to_chunks(t.file_text,
	   json('{ "by":"words",
			   "max":"50",
			   "overlap":"0",
			   "split":"recursively",
			   "language":"american",
			   "normalize":"all" }')) c
	-- joins
	where l.id = t.id
	and rownum < 11;	
    </copy>
    ```

## Task 12: Sampling of Search Queries

As the user **SEARCH23AI**, issue the below SQL, PL/SQL Code.

1. Select AI Natural Language to SQL

    ```
    <copy>
    SELECT DBMS_CLOUD_AI.GENERATE(
        prompt       => 'What movies are tom hanks in',
        profile_name => 'OCI_LLAMA_INSTRUCT_NL2SQL',
        action       => 'narrate');
    </copy>
    ```

2. SQL Statements Using Database Function AI_SEARCH Created in Task 9, Step 5

    ```
    <copy>
	set serveroutput on
    select ai_search('What movies are tom hanks in','Movies');
    select ai_search('what is the broadband infraxtetructure fund outlined in house bill 9?','Default');	
    </copy>
    ```

3. SQL Statements Using Database Function AI_SEARCH Created in Task 9, Step 10

    ```
    <copy>
	set serveroutput on
    -- Image Search
    select document_search('This image shows winners of the Canadian Squash Championships for both open and 40+.  Who won the open in 2008?','Squash.png');
    -- Document Search
    select document_search('summarize this document','constitution.pdf');
    </copy>
    ```

4. This performs a Ubiquitous Search.

    ```
    <copy>
    -- dbms_search (ubiquitous search)
    select s.source,d.file_name,  s.score, v.chunk_txt
    from documents d, document_vector v,
    (SELECT a.source, SCORE(1) score, json_value(a.key,'$.ID' returning number) key
    from global_idx a WHERE CONTAINS(a.data, 'invasion', 1) > 0 and a.source='DOCUMENTS') s
    where d.id = s.key
    and d.id = v.id
    and instr(lower(v.chunk_txt),lower('invasion'))>0
    order by s.score desc;
    -- Hybrid Search
    select h.id
        , h.file_name
        , v.vector_score
        , v.text_score
    from documents_hybrid_search h
        , (select d.id
            , d.vector_score
            , d.text_score
            from
            (select dbms_hybrid_vector.search(
        json(
          '{
             "hybrid_index_name" : "documents_hybrid_search_idx",
             "search_scorer"     : "rsf",
             "search_fusion"     : "UNION",
             "vector":
              {
                 "search_text"   : "invasion",
                 "search_mode"   : "DOCUMENT",
                 "aggregator"    : "MAX",
                 "score_weight"  : 1,
                 "rank_penalty"  : 5
              },
             "text":
              {
                 "contains"      : "invasion",
                 "score_weight"  : 10,
                 "rank_penalty"  : 1
              },
             "return":
              {
                 "values"        : [ "rowid", "score", "vector_score", "text_score" ],
                 "topN"          : 3
                    }
                }'
                )) data) j,
            json_table ( j.data, '$[*]'
                columns (
                    id varchar2(100) path '$."rowid"',
                    vector_score number path '$."vector_score"',
                    text_score number path '$."text_score"'
                            )
                        )
                    d) v
    where h.rowid = v.id
    order by v.text_score desc, v.vector_score desc;
    </copy>
    ```
	
5. SQL Statement Using Vector Distance 

    ```
    <copy>
	select  l.id, l.file_name, lv.chunk_id, 
			vector_distance(lv.embed_vector, (select dbms_vector.utl_to_embedding('what is the broadband infrastructure fund outlined in house bill 9?', json('{
				  "provider": "OCIGenAI",
				  "credential_name": "{update with oci_vector_cred from Task 3.3}",
				  "url": "https://inference.generativeai.{region}.oci.oraclecloud.com/20231130/actions/embedText",
				  "model": "cohere.embed-english-v3.0",
				  "batch_size":10}')) from dual),cosine) as score,
			lv.chunk_txt 
	from    document_vector lv, documents l
	where   l.id = lv.id and l.domain = 'Default'
			order by score
	fetch first 5 rows only;
    </copy>
    ```

6. Sample Response from above SQL Statement from Step 1

	```
	<copy>
	-- response to above prompts
	{
		"modelId": "cohere.command-r-plus",
		"modelVersion": "1.2",
		"chatResponse": {
			"apiFormat": "COHERE",
			"text": "Yes, there are financial implications associated with the passage of this bill. The bill mentions cost savings accruing to the state and an estimate of the total amount of funding as part of the reporting requirements. It also mentions a financial contribution from parents, which the commissioner and grant recipients are prohibited from requiring. However, the specific financial implications would depend on various factors, including the program's scope, the number of grant recipients, and the allocation of funds.",
			"chatHistory": [
				{
					"role": "USER",
					"message": "1, 2024, if theconstitutional amendment proposed by the 88th Legislature, RegularSession, 2023, creating the broadband infrastructure fund to expandhigh-speed broadband access and assist in the financing ofconnectivity projects in the state takes effect. AA If that amendment88(R) HB 9 - Enrolled versionH.B. ANo. A9AN ACTrelating to the development and funding of broadband andtelecommunications services.BE IT ENACTED BY THE LEGISLATURE OF THE STATE OF TEXAS:SECTION A1. AA ChapterSec. A403.603. AA BROADBAND INFRASTRUCTURE FUND. (a) Thebroadband infrastructure fund is a special fund in the statetreasury outside the general revenue fund. The fund consists of:(1) AA money transferred or deposited to the credit offund is in furtherance of the publicpurpose of expanding and ensuring access to reliable, high-speed1234567891011121314151617181920212223241broadband and telecommunications connectivity.one-timetransfer in the amount of $75 million from the broadbandinfrastructure fund established under Section 49-d-16, ArticleIII, Texas Constitution, to the broadband pole replacement fundestablished under Section 403.502, Government Code, as added by Question: what is the broadband infraxtetructure fund outlined in house bill 9?"
				},
				{
					"role": "CHATBOT",
					"message": "The broadband infrastructure fund outlined in House Bill 9 is a special fund in the state treasury of Texas, established to expand and ensure access to reliable, high-speed broadband and improve telecommunications services."
				},
				{
					"role": "USER",
					"message": "of cost savings accruing to the state asa result of the program;(6) AA in a report submitted in an even-numbered yearonly, an estimate of the total amount of fundingcommissioner and any grant recipientselected under this section may not require any financialcontribution from parents to implement and administer the grant.(h) AA AthisAct does not receive the vote necessary for immediate effect, thisAct takes effect September 1, 2023.1234567891011121314151617181920this sectionmay, subject to available funding and the requirements of thissubchapter, participate in the program until the earliest of thefollowing dates:(1) AA thewith Section 29.357(2);(4) AA the effect of the program on public and privateschool capacity, availability, and quality;(5) AA the amount of cost savings accruing to Question: are there any financial implications if it is passed?"
				},
				{
					"role": "CHATBOT",
					"message": "Yes, there are financial implications associated with the passage of this bill. The bill mentions cost savings accruing to the state and an estimate of the total amount of funding as part of the reporting requirements. It also mentions a financial contribution from parents, which the commissioner and grant recipients are prohibited from requiring. However, the specific financial implications would depend on various factors, including the program's scope, the number of grant recipients, and the allocation of funds."
				}
			],
			"finishReason": "COMPLETE"
		}
	}		
	</copy>
	```

7. Perform Search or instruction of specific document

	*** NOTE CHANGE THE DOCUMENT NAME TO MATCH ONE THAT IS IN YOUR DOCUMENTS TABLE ***

    ```
    <copy>
	set serveroutput on
	select gen_ai_chat_documents_single('Nutritian.txt','prepare a typical menu for the day based in the following nutritianal guidelines that specify a minimum of 160g of protein')	
    </copy>
    ```

    ```
    <copy>
	"A typical menu for the day based on the nutritional guidelines that specify a minimum of 160g of protein could include:\n\n* Breakfast: 3 large eggs (18g protein), 2 cups mixed berries (2g protein), 1 cup Greek yogurt (20g protein), and 1 scoop whey protein powder (30g protein) - Total: 70g protein\n* Snack: 1 cup cottage cheese (28g protein) and 1 cup sliced peaches (2g protein) - Total: 30g protein\n* Lunch: 6 oz grilled chicken breast (42g protein), 1 cup cooked quinoa (8g protein), and 1 cup steamed broccoli (4g protein) - Total: 54g protein\n* Snack: 1 cup Greek yogurt (20g protein) and 1/2 cup mixed nuts (10g protein) - Total: 30g protein\n* Dinner: 6 oz grilled salmon (40g protein), 1 cup cooked brown rice (5g protein), and 1 cup sautéed spinach (3g protein) - Total: 48g protein\n\nTotal daily protein intake: 192g\n\nThis menu provides a balanced mix of protein, healthy fats, and complex carbohydrates, and stays within the recommended daily intake of protein. However, the specific nutritional needs of an individual may vary depending on factors such as age, sex, weight, and activity level. It's always best to consult with a healthcare professional or registered dietitian to determine the best meal plan for your individual needs."
    </copy>
    ```

## Task 13: APEX Application

1.  Create New APEX Workspace

	Access Oracle APEX Administration Services from the below link 
	
	* [Access Oracle APEX Administration Services](https://docs.oracle.com/en/cloud/paas/autonomous-database/serverless/adbsb/apex-access-admin-services.html)

	Log into the **internal** Workspace as the user **admin** and corresponding password.
	
	Create a new Workspace using an existing Schema **SEARCH23AI**

	* [Creating a Workspace Manually](https://docs.oracle.com/en/database/oracle/apex/24.2/htmig/creating-workspace-and-adding-apex-users.htm)

2. Log into the new Workspace and Import the APEX Application
    
	The APEX Application is available from the below link.  You can import the app into an existing APEX Workspace based off of the **SEARCH23AI** schema/user.

	* [Download APEX Application](https://github.com/snicholspa/tips_tricks_howtos/blob/main/autonomous_database/search23ai/files/apex_search23ai_app.sql)

	* [Importing an Application Export](https://docs.oracle.com/en/database/oracle/apex/24.2/htmdb/importing-export-files.htm)

3. Setup and Enable ORDS REST access to AI Search (optional)

    As the user **SEARCH23AI**, issue the below SQL, PL/SQL Code.

    REST Enable List Files

    ```
    <copy>
    BEGIN
      ORDS.ENABLE_SCHEMA(
          p_enabled             => TRUE,
          p_schema              => 'SEARCH23AI',
          p_url_mapping_type    => 'BASE_PATH',
          p_url_mapping_pattern => 'search23ai',
          p_auto_rest_auth      => FALSE);
        
      ORDS.DEFINE_MODULE(
          p_module_name    => 'search',
          p_base_path      => '/search/',
          p_items_per_page => 25,
          p_status         => 'PUBLISHED',
          p_comments       => NULL);

      ORDS.DEFINE_TEMPLATE(
          p_module_name    => 'search',
          p_pattern        => 'listfiles',
          p_priority       => 0,
          p_etag_type      => 'HASH',
          p_etag_query     => NULL,
          p_comments       => NULL);

      ORDS.DEFINE_HANDLER(
          p_module_name    => 'search',
          p_pattern        => 'listfiles',
          p_method         => 'GET',
          p_source_type    => 'json/collection',
          p_items_per_page => 25,
          p_mimes_allowed  => NULL,
          p_comments       => NULL,
          p_source         => 'select file_name from documents');
    
    COMMIT;

    END;
    /
    </copy>
    ```

    REST Enable AI Search Function
    
    ```
    <copy>
    BEGIN
      ORDS.DEFINE_TEMPLATE(
          p_module_name    => 'search',
          p_pattern        => 'searchai/:prompt/:domain',
          p_priority       => 0,
          p_etag_type      => 'HASH',
          p_etag_query     => NULL,
          p_comments       => NULL);
          
      ORDS.DEFINE_HANDLER(
          p_module_name    => 'search',
          p_pattern        => 'searchai/:prompt/:domain',
          p_method         => 'GET',
          p_source_type    => 'json/item',
          p_items_per_page =>  25,
          p_mimes_allowed  => '',
          p_comments       => NULL,
          p_source         => 'select ai_search(:prompt,:domain)');

    COMMIT;

    END;
    /
    </copy>
    ```

    REST Enable Single Document Search
    
    ```
    <copy>
    BEGIN   
      ORDS.DEFINE_TEMPLATE(
          p_module_name    => 'search',
          p_pattern        => 'searchai2/:filename/:prompt',
          p_priority       => 0,
          p_etag_type      => 'HASH',
          p_etag_query     => NULL,
          p_comments       => NULL);

      ORDS.DEFINE_HANDLER(
          p_module_name    => 'search',
          p_pattern        => 'searchai2/:filename/:prompt',
          p_method         => 'GET',
          p_source_type    => 'json/item',
          p_items_per_page =>  25,
          p_mimes_allowed  => '',
          p_comments       => NULL,
          p_source         => 'select document_search(:prompt,:filename)');

    COMMIT;
          
    END;
    /
    </copy>
    ```

3. Test ORDS REST access to AI Search (optional)

    Enter the following URLs into a browser.
    
    List files in the documents table
    ```
    <copy>
    https://{database_name}.adb.{region}.oraclecloudapps.com/ords/search23ai/search/listfiles
    </copy>
    ```

    List files in the documents table
    ```
    <copy>
    https://{database_name}.adb.{region}.oraclecloudapps.com/ords/search23ai/search/searchai/what+is+the+broadband+infraxtetructure+fund+outlined+in+house+bill+9/Default
    </copy>
    ```

    List files in the documents table
    ```
    <copy>
    https://https://{database_name}.adb.{region}.oraclecloudapps.com/ords/search23ai/search/listfiles
    </copy>
    ```

## Acknowledgements
  * **Authors:** Derrick Cameron and Steven Nichols, Master Principal Cloud Architects
  * **Last Updated By/Date:** Steven Nichols, July 18, 2025

Copyright (C)  Oracle Corporation.

Permission is granted to copy, distribute and/or modify this document
under the terms of the GNU Free Documentation License, Version 1.3
or any later version published by the Free Software Foundation;
with no Invariant Sections, no Front-Cover Texts, and no Back-Cover Texts.
A copy of the license is included in the section entitled [GNU Free Documentation License](files/gnu-free-documentation-license.txt)