# temp-code


retStr = stringbuilder();
//prof = stringbuilder(); //Commented : 01 April 2026 : Bibin Sajeevan : Moved to PS Sub Lib
priceDataJson = json();
tableDetailsJson = json();
if(finalPriceJson_t_c <> ""){
    priceDataJson = json(finalPriceJson_t_c);
}

inputJson = json();
jsonput(inputJson,"tableName","ProposalData");
tableDetailsJson = json();
jsonput(inputJson,"skuArr",allSKUNumbers_t_c);
tableDetailsJson = util.util_dataTableJson(inputJson);

inputJson = json();
jsonput(inputJson,"tableName","CustSKUPropTempData");
customTableDetailsJson = json();
jsonput(inputJson,"skuArr",allSKUNumbers_t_c);
customTableDetailsJson = util.util_dataTableJson(inputJson);

modelsNameArr = string[];

allModelsJson = json();
if(allModels_t_c <> ""){
    allModelsJson = json(allModels_t_c);
    modelsNameArr = jsonkeys(allModelsJson);
}

modelsSeq = bmql("select ModelVarName,SequenceNum,RootModelName from CoreProducts where ModelVarName IN $modelsNameArr ORDER BY SequenceNum");
modelsArr = string[];
modelsNameArr = string[];
//Added: Bibin Sajeevan : 19 Nov 2025 : Json to store abstract models (FPO model)
pseudoModelJson = json(); 
definiteModelsArr = string[];
abstractModelsJson = json();
abstractModelsDocJson = json();
//Added: Bibin Sajeevan : 19 Nov 2025 : Json to store abstract models (FPO model)
for model in modelsSeq{
    modelDetailsJson = jsonget(allModelsJson,get(model,"ModelVarName"),"json",json());
    docNum = jsonget(modelDetailsJson,"docNum");
    append(modelsNameArr,get(model,"ModelVarName"));
    append(modelsArr,docNum);
    //Added: Bibin Sajeevan : 20 Nov 2025 : Collecting abstract models
    if(get(model,"RootModelName") <> ""){
        jsonput(abstractModelsJson,get(model,"ModelVarName"),get(model,"RootModelName"));
        jsonput(abstractModelsDocJson,docNum,get(model,"RootModelName"));
    }
    else{
        append(definiteModelsArr,docNum);
    }
    //Added: Bibin Sajeevan : 20 Nov 2025 : Collecting abstract models
}
totalModels = sizeofarray(modelsArr);

//retStr = sbappend(retStr,"1~proposalModelSequence_t_c~",join(modelsArr,"#_#"),"|"); //Commented : Bibin Sajeevan : 20 November 2025
//retStr = sbappend(retStr,"1~proposalModelSequence_t_c~",join(definiteModelsArr,"#_#"),"|");
setattributevalue(1, "proposalModelSequence_t_c", join(definiteModelsArr,"#_#"));
modelArrSize = sizeofarray(modelsArr) + 1;
//lineFields = string[]{"_part_number","_sequence_number","_model_is_valid","_part_desc","_line_item_type","_line_bom_level","_document_number","listAmount_l","totalRebatefeeNetPriceRollup_l_c","_model_name","requestedUnitOfMeasure_l","_price_quantity","listPrice_l","netPrice_l","netAmount_l","feeType_l_c","parentDocNumber_l","billingFrequency_l_c","calculationExclusionFlag_l_c","_line_item_type","rollupSKU_l_c","oRCL_ABO_ActionCode_l","minimumCalcGrpSKUID_l_c","grpMinSKUDetails_l_c","aBOJson_l_c","qtyFactor_l_c","feeCalculationMethod_l_c","_model_variable_name","displayNetPrice_l_c","customDiscountValue_l","titleOnProposal_l_c","minimumDiscount_l_c","listFeeMinimumDisplay_l_c"};Commented on 12MAY25 by Ashok

//lineFields = string[]{"_part_number","_sequence_number","_model_is_valid","_part_desc","_line_item_type","_line_bom_level","_document_number","listAmount_l","displayListPrice_l_c","_model_name","requestedUnitOfMeasure_l","_price_quantity","listPrice_l","netPrice_l","netAmount_l","feeType_l_c","parentDocNumber_l","billingFrequency_l_c","calculationExclusionFlag_l_c","_line_item_type","rollupSKU_l_c","oRCL_ABO_ActionCode_l","minimumCalcGrpSKUID_l_c","grpMinSKUDetails_l_c","aBOJson_l_c","qtyFactor_l_c","_model_variable_name","displayNetPrice_l_c","customDiscountValue_l","titleOnProposal_l_c","minimumDiscount_l_c","listFeeMinimumDisplay_l_c"}; //Commented : 02 June 2025 : Using custom action code attribute
lineFields = string[]{"_part_number","_sequence_number","_model_is_valid","_part_desc","applicationName_l_c","_line_item_type","_line_bom_level","_document_number","listAmount_l","displayListPrice_l_c","_model_name","requestedUnitOfMeasure_l","_price_quantity","listPrice_l","netPrice_l","netAmount_l","feeType_l_c","parentDocNumber_l","billingFrequency_l_c","calculationExclusionFlag_l_c","_line_item_type","rollupSKU_l_c","minimumCalcGrpSKUID_l_c","grpMinSKUDetails_l_c","aBOJson_l_c","qtyFactor_l_c","_model_variable_name","displayNetPrice_l_c","customDiscountValue_l","titleOnProposal_l_c","minimumDiscount_l_c","listFeeMinimumDisplay_l_c","customActionCode_l_c","isGroupMinimumContributor_l_c","aCVExclusionFlag_l_c","formulaInputSKUs_l_c","minimumFeeApplied_l_c","aggregatedProducts_l_c","descriptionOnProposal_custom_l_c","typeOfTier_custom_l_c","presentationType_l_c"};
modelDocNum = "";
modelName = "";
modelVarName = "";
indentationStr = "";
transactionPriceTableJson = json();
relationshipCreditsJson = json();
modelJson = json();
custConfigDataJson = json();
lvl1PriceTablesJson = json();
parentOptionClassJson = json();
getBOMDetailsJson = json();
lineDetailsJson = json();
displayNameJson = json();
qrtDisplayNameJson = json();
inclusionsJson = json();
childSKUsJson = json();
qrtFlags = string[]{"QRT","QRT1","QRT2","QRT3"};
tierIndentationJson = json(applytemplate("$BASE_PATH$/TiersIndentation.txt"));
counter = 1;
childSKUCounter = 0;
noOfChildSKUs = -1;
activeSKUNumbers = stringbuilder();
assumptionDataJson = json();
lineDataJson = json();
abstractModelLineDataJson = json();

//Added by FIS Team : 21 August 2025
//titleOnProposal = ""; //Commented : 26 March 2026 : Bibin Sajeevan : Moved to PS Sub Lib
netAmountFixedFee = 0.00;
//fixedFeeJson = json(); //Commented : 26 March 2026 : Bibin Sajeevan : Moved to PS Sub Lib
fixedFeeMergeJson = json();
//Added by FIS Team : 21 August 2025
activeSKUCountJson = json();

for line in transactionline
{
    //#################### Collecting data for Assumption Data Json############################
    //if(line.customActionCode_l_c <> "NO_UPDATE" AND line.customActionCode_l_c <> "DELETE" AND line._part_number <> ""){
    if((line.customActionCode_l_c <> "NO_UPDATE" AND line.customActionCode_l_c <> "DELETE") OR line._model_is_valid){
        activeSKUNumbers = sbappend(activeSKUNumbers,line._part_number,"##");
        partDetailsJson = json();
        jsonput(partDetailsJson,"createdDate_l",line.createdDate_l);
        if(line._model_is_valid){
            jsonput(lineDataJson,line._model_variable_name,partDetailsJson);
            //Added: Bibin Sajeevan : 20 Nov 2025 : Collecting abstract models Details
            if(NOT isnull(jsonget(abstractModelsJson,line._model_variable_name))){
                abstractModelName = jsonget(abstractModelsJson,line._model_variable_name);
                abstractModelDetailsJson = jsonget(abstractModelLineDataJson,abstractModelName ,"json",json());
                if(NOT isnull(jsonget(abstractModelDetailsJson,"createdDate_l"))){
                    previousCreatedDate = strtojavadate(jsonget(abstractModelDetailsJson,"createdDate_l","string",""), "yyyy-MM-dd HH:mm:ss");
                    currentCreatedDate =  strtojavadate(line.createdDate_l,"yyyy-MM-dd HH:mm:ss");
                    if(comparedates(previousCreatedDate,currentCreatedDate) == 1){
                        jsonput(abstractModelDetailsJson,"createdDate_l",line.createdDate_l);
                    }
                }
                else{
                    jsonput(abstractModelDetailsJson,"createdDate_l",line.createdDate_l);
                }
                jsonput(abstractModelLineDataJson ,abstractModelName,abstractModelDetailsJson);
            }
            //Added: Bibin Sajeevan : 20 Nov 2025 : Collecting abstract models Details
        }
        else{
            jsonput(lineDataJson,line._part_number,partDetailsJson);
            if(line.modelDetails_l_c <> ""){
                modelVarName = jsonget(json(line.modelDetails_l_c),"modelVarName","string","");
                jsonput(activeSKUCountJson,modelVarName,jsonget(activeSKUCountJson,modelVarName,"integer",0)+1);
            }
        }
    }
    //#################### Collecting data for Assumption Data Json############################

    if(line.feeType_l_c == "Rebate" AND line.parentDocNumber_l == ""){
        relationshipCreditTableRowStr = stringbuilder();
        rCMonthly = 0;
        rCOneTime = 0;
        if(line.billingFrequency_l_c == "Monthly"){
            rCMonthly = line.netAmount_l;
        }
        elif(line.billingFrequency_l_c == "One Time"){
            rCOneTime = line.netAmount_l;
        }
        sbappend(relationshipCreditTableRowStr,"@_@",line.sKUDisplayName_l_c,"*_*",string(rCMonthly),"*_*","0","*_*",string(rCOneTime));
        if(line.calculationExclusionFlag_l_c <> "y"){
            jsonput(relationshipCreditsJson,"relationshipCreditMonthlyTotal",jsonget(relationshipCreditsJson,"relationshipCreditMonthlyTotal","float",0.0)+rCMonthly);
            jsonput(relationshipCreditsJson,"relationshipCreditOneTimeTotal",jsonget(relationshipCreditsJson,"relationshipCreditOneTimeTotal","float",0.0)+rCOneTime);
        }
        jsonput(relationshipCreditsJson,"relationshipCreditStr",jsonget(relationshipCreditsJson,"relationshipCreditStr","string","")+sbtostring(relationshipCreditTableRowStr));
        continue;
    }
    
    modelLine = false;
    if(childSKUCounter == noOfChildSKUs){
        remove(modelsArr,0);
        remove(modelsNameArr,0);
        childSKUCounter = 0;
    }
    
    if(sizeofarray(modelsArr) < modelArrSize){
        modelLine = true;
        currentModelDocNum = modelsArr[0];
        currentModelName = modelsNameArr[0];
        modelArrSize  = sizeofarray(modelsArr);
        getBOMDetailsJson = getbom(atoi(bs_id),atoi(currentModelDocNum),lineFields,false,true);
        getBOMChildSKUs = jsonget(getBOMDetailsJson,"children","jsonarray",jsonarray());
        noOfChildSKUs = jsonarraysize(getBOMChildSKUs) + 1;
        modelFieldsJson = jsonget(getBOMDetailsJson,"fields","json",json());
        counter = jsonget(modelFieldsJson,"_sequence_number","integer",0);
    }
    if(modelLine){
        lineDetailsJson = jsonpathgetsingle(getBOMDetailsJson,"$.fields","json",json());
    }
    else{
        lineDetailsJson = jsonpathgetsingle(getBOMDetailsJson,"$.children[?(@.fields._sequence_number == '"+string(counter)+"')].fields","json",json());
    }
    partNumber = jsonget(lineDetailsJson,"_part_number");
    bomLevel = jsonget(lineDetailsJson,"_line_bom_level");
    proposalDataJson = jsonget(tableDetailsJson,partNumber,"json",json());
    counter = counter + 1;
    childSKUCounter = childSKUCounter + 1;
    
    if(jsonget(lineDetailsJson,"_model_is_valid","boolean",false) AND jsonget(lineDetailsJson,"parentDocNumber_l","string","") == ""){ 
        inputJson = json();
        jsonput(inputJson,"parentOptionClassJson",parentOptionClassJson);
        jsonput(inputJson,"lvl1PriceTablesJson",lvl1PriceTablesJson);
        reponse = commerce.updatePreviousOptionClass_c(inputJson);
        lvl1PriceTablesJson = jsonget(reponse,"lvl1PriceTablesJson","json",lvl1PriceTablesJson);
        parentOptionClassJson = jsonget(reponse,"parentOptionClassJson","json",parentOptionClassJson);
        
	//Added: Bibin Sajeevan : 01 April 2026 : Fixed Fee Merge Sub Lib : QTCCPQ - 4455
	if(sizeofarray(jsonkeys(fixedFeeMergeJson)) > 0){
	    inputJson = json();
	    jsonput(inputJson,"fixedFeeMergeJson",fixedFeeMergeJson);
	    jsonput(inputJson,"lvl1PriceTablesJson",lvl1PriceTablesJson);
	    response = commerce.mergePSFixedFees_c(inputJson);
	    lvl1PriceTablesJson = jsonget(response,"lvl1PriceTablesJson","json",lvl1PriceTablesJson);
	    fixedFeeMergeJson = json();
	}
	//Added: Bibin Sajeevan : 01 April 2026 : Fixed Fee Merge Sub Lib : QTCCPQ - 4455
        
        //Added : Bibin Sajeevan : 24 February 2026 : Custom SKU Config
        if(NOT isnull(jsonget(custConfigDataJson,jsonget(lvl1PriceTablesJson,"tableHeader","string","")))){
            customSKUsDataJson = jsonget(custConfigDataJson,jsonget(lvl1PriceTablesJson,"tableHeader","string",""),"json",json());
            inputJson = json();
            jsonput(inputJson,"modelJson",modelJson);
            jsonput(inputJson,"lvl1PriceTablesJson",lvl1PriceTablesJson);
            jsonput(inputJson,"customSKUsDataJson",customSKUsDataJson);
            jsonput(inputJson,"tierIndentationJson",tierIndentationJson);
            reponse = commerce.mergeCustomSKUDetails_c(inputJson);
            jsonremove(custConfigDataJson,jsonget(lvl1PriceTablesJson,"tableHeader","string",""));
            lvl1PriceTablesJson = jsonget(reponse,"lvl1PriceTablesJson","json",lvl1PriceTablesJson);
            modelJson = jsonget(reponse,"modelJson","json",modelJson);
        }
        //Added : Bibin Sajeevan : 24 February 2026 : Custom SKU Config
        
        inputJson = json();
        jsonput(inputJson,"lvl1PriceTablesJson",lvl1PriceTablesJson);
        jsonput(inputJson,"lineDetailsJson",lineDetailsJson);
        jsonput(inputJson,"transactionPriceTableJson",transactionPriceTableJson);
        jsonput(inputJson,"modelJson",modelJson);
        //Added: Bibin Sajeevan : 20 Nov 2025 : Abstract Model Specific Conditions
        if(NOT isnull(jsonget(abstractModelsJson,jsonget(modelJson,"modelVarName","string","")))){
            jsonput(inputJson,"abstractModel",jsonget(abstractModelsJson,jsonget(modelJson,"modelVarName","string","")));
            jsonput(inputJson,"pseudoModelJson",pseudoModelJson);
        }
        //Added: Bibin Sajeevan : 20 Nov 2025 : Abstract Model Specific Conditions
        reponse = commerce.collectLvl1OptionClassDetails_c(inputJson); 
        lvl1PriceTablesJson = json();
        pseudoModelJson = jsonget(reponse,"pseudoModelJson","json",pseudoModelJson);
        modelJson = jsonget(reponse,"modelJson","json",modelJson);
        transactionPriceTableJson = jsonget(reponse,"transactionPriceTableJson","json",transactionPriceTableJson);
        
        //###### Adding new Tables if Custom SKU Config table is not present in the Transaction ########
	newCustTableKeys = jsonkeys(custConfigDataJson);
	if(sizeofarray(newCustTableKeys) > 0){
		for newTableHeader in newCustTableKeys{
		    customSKUsDataJson = jsonget(custConfigDataJson,newTableHeader,"json",json());
		    inputJson = json();
		    jsonput(inputJson,"modelJson",modelJson);
		    jsonput(inputJson,"lvl1PriceTablesJson",lvl1PriceTablesJson);
		    jsonput(inputJson,"customSKUsDataJson",customSKUsDataJson);
		    jsonput(inputJson,"tierIndentationJson",tierIndentationJson);
		    reponse = commerce.mergeCustomSKUDetails_c(inputJson);
		    jsonremove(custConfigDataJson,newTableHeader);
		    lvl1PriceTablesJson = jsonget(reponse,"lvl1PriceTablesJson","json",lvl1PriceTablesJson);
		    modelJson = jsonget(reponse,"modelJson","json",modelJson);
		    
		    inputJson = json();
		    jsonput(lvl1PriceTablesJson,"tableHeader",newTableHeader);
		    jsonput(inputJson,"lvl1PriceTablesJson",lvl1PriceTablesJson);
		    jsonput(inputJson,"transactionPriceTableJson",transactionPriceTableJson); 
		    jsonput(inputJson,"modelJson",modelJson);
		    if(NOT isnull(jsonget(abstractModelsJson,jsonget(modelJson,"modelVarName","string","")))){
		        jsonput(inputJson,"abstractModel",jsonget(abstractModelsJson,jsonget(modelJson,"modelVarName","string","")));
		        jsonput(inputJson,"pseudoModelJson",pseudoModelJson);
		    } 
		    reponse = commerce.collectLvl1OptionClassDetails_c(inputJson);
		    lvl1PriceTablesJson = json();
		    modelJson = jsonget(reponse,"modelJson","json",modelJson);
		    pseudoModelJson = jsonget(reponse,"pseudoModelJson","json",pseudoModelJson);
		    transactionPriceTableJson = jsonget(reponse,"transactionPriceTableJson","json",transactionPriceTableJson);
		}
	}
    	//###### Adding new Tables if Custom SKU Config table is not present in the Transaction ########
        
        //######### Updating previous model data ########### 
        if(modelDocNum <> ""){
            inputJson = json();
            jsonput(inputJson,"modelJson",modelJson);
            jsonput(inputJson,"transactionPriceTableJson",transactionPriceTableJson);
            if(NOT isnull(jsonget(abstractModelsJson,jsonget(modelJson,"modelVarName","string","")))){
                jsonput(inputJson,"abstractModel",jsonget(abstractModelsJson,jsonget(modelJson,"modelVarName","string","")));
                jsonput(inputJson,"pseudoModelJson",pseudoModelJson);
            }
            reponse = commerce.collectModelDetails_c(inputJson);
            pseudoModelJson = jsonget(reponse,"pseudoModelJson","json",pseudoModelJson);
            transactionPriceTableJson = jsonget(reponse,"transactionPriceTableJson","json",transactionPriceTableJson);
            sbappend(retStr,jsonget(reponse,"retStr","string",""));
        }
        //######### Updating previous model data ##########
        modelJson = json();
        modelDocNum = jsonget(lineDetailsJson,"_document_number");
        jsonput(modelJson,"modelName",jsonget(lineDetailsJson,"_model_name"));
        jsonput(modelJson,"modelVarName",jsonget(lineDetailsJson,"_model_variable_name"));
        jsonput(modelJson,"modelDocNum",modelDocNum);
        if(jsonget(modelJson,"modelVarName","string","") == "professionalServices_m"){
            jsonput(lvl1PriceTablesJson,"lvl1OpClassDocNum",modelDocNum);
        }
        if(NOT(isnull(getconfigattrvalue(modelDocNum, "bSol_dynamicDisplayNameJson_pf"))) AND getconfigattrvalue(modelDocNum, "bSol_dynamicDisplayNameJson_pf") <> ""){
            dynamicDisplayJson = json(getconfigattrvalue(modelDocNum,"bSol_dynamicDisplayNameJson_pf"));
            displayNameJson = jsonget(dynamicDisplayJson,"displayNameJson","json",json());
            if(isnull(jsonget(dynamicDisplayJson,"displayNameJson"))){
                displayNameJson = dynamicDisplayJson;
            }
            qrtDisplayNameJson = jsonget(dynamicDisplayJson,"qrtDisplayNameJson","json",json()); 
            inclusionsJson = jsonget(dynamicDisplayJson,"inclusionsJson","json",json());
        }
    }
    elif(jsonget(lineDetailsJson,"_model_is_valid","boolean",false) AND jsonget(lineDetailsJson,"parentDocNumber_l","string","") <> ""){
        jsonput(modelJson,"customConfigDoc",jsonget(lineDetailsJson,"_document_number","string",""));
    }
    
    currentPriceDataJson = jsonget(priceDataJson,partNumber+"**"+modelDocNum,"json",json());
    priceTableRowStr = stringbuilder();
    rowPricesJson = json();
    qrtSKU = "";
    skuDisplayName = jsonget(proposalDataJson,"SKUDisplay","string","");
    altSkuDisplayName = "";
    skuDisplayName = jsonget(displayNameJson,partNumber,"string",skuDisplayName);
    skuDisplayName = replace(replace(replace(skuDisplayName,"&","&amp;"),"<","&lt;"),">","&gt;");
    nonIndentSkuDisplayName = skuDisplayName;
    rowSecSep = "@_@";
        
    //################ Calculating Indentation ############
    indentArrSize = jsonget(proposalDataJson,"Indentation","integer",0);
    totalIndentSize = range(indentArrSize);
    indentationStr = "";
    for indent in totalIndentSize{
        indentationStr = indentationStr + "&#160;&#160;&#160;";
    }
    //################ Calculating Indentation ############
    
    //################ Calculating Fee Table Indentation ############
    feeTableIndentArrSize = jsonget(proposalDataJson,"FeeTableIndentation","integer",0);
    feeTableTotalIndentSize = range(feeTableIndentArrSize);
    feeTableIndentationStr = "";
    for indent in feeTableTotalIndentSize{
        feeTableIndentationStr = feeTableIndentationStr + "&#160;&#160;&#160;";
    }
    //################ Calculating Fee Table Indentation ############

    if(jsonget(proposalDataJson,"SectionId","string","") <> ""){
        rowSecSep = "!_!";
        if(jsonget(proposalDataJson,"SectionId","string","") <> jsonget(lvl1PriceTablesJson,"SectionId","string","")){
            rowSecSep = "@_@";
        }
    }
    jsonput(lvl1PriceTablesJson,"SectionId",jsonget(proposalDataJson,"SectionId","string",""));
    
    if(jsonget(lineDetailsJson,"_line_item_type") == "Option Class"){
        jsonput(lvl1PriceTablesJson,"currentOptionClass",jsonget(lineDetailsJson,"_part_number"));
        jsonput(lvl1PriceTablesJson,"displayStdSKU",false);
        
        //################ Getting the actual SKU Display Name ##############
        actionCodeCheckStr = ""; 
        //if(transactionType_t_c == "Amendment" OR transactionType_t_c == "Renewal"){
        if(transactionType_t_c <> "New"){
            //actionCodeCheckStr = "&& @.fields.oRCL_ABO_ActionCode_l != 'DELETE' && @.fields.oRCL_ABO_ActionCode_l != 'NO_UPDATE'"; //Commented : 02 June 2025 : Using custom action code attribute
            actionCodeCheckStr = "&& @.fields.customActionCode_l_c != 'DELETE' && @.fields.customActionCode_l_c != 'NO_UPDATE'";
        }
        
        childSKUsJsonArray = jsonpathgetmultiple(getBOMDetailsJson,"$.children[?(@.fields.parentDocNumber_l == '"+jsonget(lineDetailsJson,"_document_number")+"' "+actionCodeCheckStr+")].fields");
        
        actualSKUTableJson = json();
        jsonput(childSKUsJson,"arrayPacket",childSKUsJsonArray);
        actualSKUNumber = "";
        //calculationExclusionFlag = "&& (@.calculationExclusionFlag_l_c != 'y' || (@.minimumCalcGrpSKUID_l_c != 'NA' && @.minimumCalcGrpSKUID_l_c != ''))"; // Updated : Bibin Sajeevan : 22 December 2025 : Skipping aggregated group minimum SKUs also // ATMNC(1330-4327-93138)
        calculationExclusionFlag = "&& (@.calculationExclusionFlag_l_c != 'y' || (@.minimumCalcGrpSKUID_l_c != 'NA' && @.minimumCalcGrpSKUID_l_c != '' && @.aggregatedProducts_l_c == ''))";

        otherFeeSKU = jsonpathgetmultiple(childSKUsJson,"$..[?(@.feeType_l_c == 'Monthly' && @.rollupSKU_l_c == '"+partNumber+"' "+calculationExclusionFlag+")]._part_number");
        implementationSKU = jsonpathgetmultiple(childSKUsJson,"$..[?(@.feeType_l_c == 'Implementation' && @.rollupSKU_l_c == '"+partNumber+"' "+calculationExclusionFlag+")]._part_number");
        licenseSKU = jsonpathgetmultiple(childSKUsJson,"$..[?(@.feeType_l_c == 'License' && @.rollupSKU_l_c == '"+partNumber+"' "+calculationExclusionFlag+")]._part_number");
        perOccurrenceSKU = jsonpathgetmultiple(childSKUsJson,"$..[?(@.feeType_l_c == 'Per Occurrence' && @.rollupSKU_l_c == '"+partNumber+"' "+calculationExclusionFlag+")]._part_number");
        //Bibin Sajeevan: 25 September 2025 : Fetching annual SKUs
        yearsSKU = jsonpathgetmultiple(childSKUsJson,"$..[?(@.feeType_l_c == 'Years' && @.rollupSKU_l_c == '"+partNumber+"' "+calculationExclusionFlag+")]._part_number");
        //Bibin Sajeevan: 25 September 2025 : Fetching annual SKUs
        rebateSKU = jsonpathgetmultiple(childSKUsJson,"$..[?(@.feeType_l_c == 'Rebate' && @.rollupSKU_l_c == '"+partNumber+"' "+calculationExclusionFlag+")]._part_number");
        
        tempParentOptionClassJson = json();
        noOfMonthlySKUs = jsonarraysize(otherFeeSKU);
	if(noOfMonthlySKUs == 1){
		jsonput(tempParentOptionClassJson,"monthlySKULineDataJson",jsonpathgetsingle(childSKUsJson,"$.arrayPacket.[?(@._part_number == '"+jsonarrayget(otherFeeSKU,0,"string")+"')]","json",json()));
		jsonput(tempParentOptionClassJson,"monthlySKUProposalDataJson",jsonget(tableDetailsJson,jsonarrayget(otherFeeSKU,0,"string"),"json",json()));
		jsonput(tempParentOptionClassJson,"monthlySKUPriceDataJson",jsonget(priceDataJson,jsonarrayget(otherFeeSKU,0,"string")+"**"+modelDocNum,"json",json()));
	}
	noOfImplementationSKUs = jsonarraysize(implementationSKU);
	if(noOfImplementationSKUs == 1){
		jsonput(tempParentOptionClassJson,"implementationSKULineDataJson",jsonpathgetsingle(childSKUsJson,"$.arrayPacket.[?(@._part_number == '"+jsonarrayget(implementationSKU,0,"string")+"')]","json",json()));
		jsonput(tempParentOptionClassJson,"implementationSKUProposalDataJson",jsonget(tableDetailsJson,jsonarrayget(implementationSKU,0,"string"),"json",json()));
		jsonput(tempParentOptionClassJson,"implementationSKUPriceDataJson",jsonget(priceDataJson,jsonarrayget(implementationSKU,0,"string")+"**"+modelDocNum,"json",json()));
	}
	noOfLicenseSKUs = jsonarraysize(licenseSKU);      
	if(noOfLicenseSKUs == 1){
		jsonput(tempParentOptionClassJson,"licenseSKULineDataJson",jsonpathgetsingle(childSKUsJson,"$.arrayPacket.[?(@._part_number == '"+jsonarrayget(licenseSKU,0,"string")+"')]","json",json()));
		jsonput(tempParentOptionClassJson,"licenseSKUProposalDataJson",jsonget(tableDetailsJson,jsonarrayget(licenseSKU,0,"string"),"json",json()));
		jsonput(tempParentOptionClassJson,"licenseSKUPriceDataJson",jsonget(priceDataJson,jsonarrayget(licenseSKU,0,"string")+"**"+modelDocNum,"json",json()));
	}
	noOfYearsSKUs = jsonarraysize(yearsSKU);
	if(noOfYearsSKUs == 1){
		jsonput(tempParentOptionClassJson,"yearsSKULineDataJson",jsonpathgetsingle(childSKUsJson,"$.arrayPacket.[?(@._part_number == '"+jsonarrayget(yearsSKU,0,"string")+"')]","json",json()));
		jsonput(tempParentOptionClassJson,"yearsSKUProposalDataJson",jsonget(tableDetailsJson,jsonarrayget(yearsSKU,0,"string"),"json",json()));
		jsonput(tempParentOptionClassJson,"yearsSKUPriceDataJson",jsonget(priceDataJson,jsonarrayget(yearsSKU,0,"string")+"**"+modelDocNum,"json",json()));
	}
	noOfPerOccurrenceSKUs = jsonarraysize(perOccurrenceSKU);
	if(noOfPerOccurrenceSKUs == 1){
		jsonput(tempParentOptionClassJson,"perOccurrenceSKULineDataJson",jsonpathgetsingle(childSKUsJson,"$.arrayPacket.[?(@._part_number == '"+jsonarrayget(perOccurrenceSKU,0,"string")+"')]","json",json()));
		jsonput(tempParentOptionClassJson,"perOccurrenceSKUProposalDataJson",jsonget(tableDetailsJson,jsonarrayget(perOccurrenceSKU,0,"string"),"json",json()));
		jsonput(tempParentOptionClassJson,"perOccurrenceSKUPriceDataJson",jsonget(priceDataJson,jsonarrayget(perOccurrenceSKU,0,"string")+"**"+modelDocNum,"json",json()));
	}
	noOfRebateSKUs = jsonarraysize(rebateSKU);
	
	jsonput(tempParentOptionClassJson,"noOfMonthlySKUs",noOfMonthlySKUs); 
        jsonput(tempParentOptionClassJson,"noOfImplementationSKUs",noOfImplementationSKUs); 
        jsonput(tempParentOptionClassJson,"noOfLicenseSKUs",noOfLicenseSKUs); 
        jsonput(tempParentOptionClassJson,"noOfPerOccurrenceSKUs",noOfPerOccurrenceSKUs);
        jsonput(tempParentOptionClassJson,"noOfRebateSKUs",noOfRebateSKUs);
        
        if(noOfMonthlySKUs == 1 AND noOfImplementationSKUs  < 2 AND  noOfLicenseSKUs < 2){
            actualSKUNumber = jsonarrayget(otherFeeSKU,0,"string");    
        }
        elif(noOfPerOccurrenceSKUs == 1 AND noOfImplementationSKUs  < 2 AND  noOfLicenseSKUs < 2){
            actualSKUNumber = jsonarrayget(perOccurrenceSKU,0,"string");
        }
        //Bibin Sajeevan : 25 September 2025 : Annual SKU count checks
        elif(noOfYearsSKUs == 1 AND noOfMonthlySKUs == 0 AND noOfImplementationSKUs < 2 AND noOfLicenseSKUs < 2){
            actualSKUNumber = jsonarrayget(yearsSKU,0,"string");
        }
        //Bibin Sajeevan : 25 September 2025 : Annual SKU count checks
        elif(noOfLicenseSKUs == 1 AND noOfMonthlySKUs == 0 AND noOfYearsSKUs == 0 AND noOfPerOccurrenceSKUs == 0 /*AND noOfTotalSKUs == 0*/ AND noOfImplementationSKUs < 2){
            actualSKUNumber = jsonarrayget(licenseSKU,0,"string");
        }
        elif(noOfImplementationSKUs == 1 AND noOfLicenseSKUs == 0 AND noOfYearsSKUs == 0 AND noOfMonthlySKUs == 0 AND noOfPerOccurrenceSKUs == 0 /*AND noOfTotalSKUs == 0*/){
            actualSKUNumber = jsonarrayget(implementationSKU,0,"string");
        }
        //Bibin Sajeevan : 25 September 2025 : Annual SKU count checks
        elif(noOfPerOccurrenceSKUs == 1){
            actualSKUNumber = jsonarrayget(perOccurrenceSKU,0,"string");
        }
        elif(noOfRebateSKUs == 1){
            actualSKUNumber = jsonarrayget(rebateSKU,0,"string");
        }
        elif(jsonpathcheck(childSKUsJson,"$..[?(@.billingFrequency_l_c == 'Monthly' && @.requestedUnitOfMeasure_l == 'Number of Branches')]") AND noOfMonthlySKUs == 2){
            actualSKUNumber = jsonpathgetsingle(childSKUsJson,"$..[?(@.billingFrequency_l_c == 'Monthly' && @.requestedUnitOfMeasure_l != 'Number of Branches')]._part_number");
            docNumber = jsonpathgetsingle(childSKUsJson,"$..[?(@._part_number == '"+actualSKUNumber+"')]._document_number");
            jsonput(rowPricesJson,"nonBranchSKU",docNumber);
        }
        elif(jsonpathcheck(childSKUsJson,"$.arrayPacket[?(@.feeType_l_c != '' && @.feeType_l_c != 'NA' && @.feeType_l_c != 'Included' && @.feeType_l_c != 'Miscellaneous' "+calculationExclusionFlag+")]._part_number")){
            jsonput(lvl1PriceTablesJson,"displayStdSKU",true);    
        }
        
        actualSKUTableJson = jsonget(tableDetailsJson,actualSKUNumber,"json",json());
        optionClassSKUDisplayName = "";
        nonIndentOptionClassSKUDisplayName = "";
        if(skuDisplayName <> ""){
            nonIndentOptionClassSKUDisplayName = skuDisplayName;
            if(jsonget(proposalDataJson,"DisplayType","string","") == "PBWGC"){
                optionClassSKUDisplayName = "\\b"+indentationStr+skuDisplayName;
            }
            else{
                optionClassSKUDisplayName = indentationStr+skuDisplayName;
            }
        }
        if(jsonget(actualSKUTableJson,"SKUDisplay","string","") <> ""){
            skuDisplayName = jsonget(actualSKUTableJson,"SKUDisplay","string","");
        }
        skuDisplayName = jsonget(displayNameJson,actualSKUNumber,"string",skuDisplayName);
        skuDisplayName = replace(replace(replace(skuDisplayName,"&","&amp;"),"<","&lt;"),">","&gt;");
        nonIndentSkuDisplayName = skuDisplayName;
        if(skuDisplayName <> ""){
            indentArrSize = jsonget(actualSKUTableJson,"Indentation","integer",0);
            totalIndentSize = range(indentArrSize);
            indentationStr = "";
            for indent in totalIndentSize{
                indentationStr = indentationStr + "&#160;&#160;&#160;";
            }
            skuDisplayName = indentationStr+skuDisplayName;
        }
        qrtSKU = jsonget(actualSKUTableJson,"DisplayFormat","string","");
        altSkuDisplayName = jsonget(actualSKUTableJson,"AltSKUDisplay","string","");
        //################ Getting the actual SKU Display Name ##############
        
        inputJson = json();
        jsonput(inputJson,"parentOptionClassJson",parentOptionClassJson);
        jsonput(inputJson,"lvl1PriceTablesJson",lvl1PriceTablesJson);
        jsonput(inputJson,"skuDisplayName",skuDisplayName);
        jsonput(inputJson,"bomLevel",bomLevel);
        reponse = commerce.updatePreviousOptionClass_c(inputJson);
        lvl1PriceTablesJson = jsonget(reponse,"lvl1PriceTablesJson","json",lvl1PriceTablesJson);
        parentOptionClassJson = jsonget(reponse,"parentOptionClassJson","json",parentOptionClassJson);
        
        //################# Print With Grandchildren ###############################
        if(NOT isnull(jsonget(lvl1PriceTablesJson,"printWithChildJsonArr"))){
            reqJson = json();
            jsonput(reqJson,"lvl1PriceTablesJson",lvl1PriceTablesJson);
            jsonput(reqJson,"lineDetailsJson",lineDetailsJson);
            lvl1PriceTablesJson = jsonget(commerce.setPrintWithGrandChild_c(reqJson),"response","json",json());
        }
        //################# Print With Grandchildren ###############################
        
        //#################### Handling No Update Option Classes #######################
        if(jsonget(proposalDataJson,"DisplayType","string","") == "PWGC" OR jsonget(proposalDataJson,"DisplayType","string","") == "PBWGC"){
            //if(optionClassSKUDisplayName <> "" AND jsonarraysize(childSKUsJsonArray) == 0 AND jsonget(lineDetailsJson,"oRCL_ABO_ActionCode_l","string","") == "NO_UPDATE"){ //Commented : 02 June 2025 : Using custom action code attribute
            if(optionClassSKUDisplayName <> "" AND jsonarraysize(childSKUsJsonArray) == 0 AND jsonget(lineDetailsJson,"customActionCode_l_c","string","") == "NO_UPDATE"){
                currentTable = jsonget(proposalDataJson,"PriceTable","string","");
                currentTableStrSize = len(jsonget(lvl1PriceTablesJson,currentTable,"string",""));
                //print jsonget(lvl1PriceTablesJson,currentTable,"string","");
                optionClassSKUDisplayNameSize = len(optionClassSKUDisplayName);
                printWithChildJsonArr = jsonget(lvl1PriceTablesJson,"printWithChildJsonArr","jsonarray",jsonarray());
                bomLevel = jsonget(lineDetailsJson,"_line_bom_level","string","");
                printWithChildJson = json();
                jsonput(printWithChildJson,"start",currentTableStrSize);
                jsonput(printWithChildJson,"table",currentTable);
                jsonput(printWithChildJson,"level",atoi(bomLevel));
                jsonput(printWithChildJson,"opClass",rowSecSep+optionClassSKUDisplayName);
                jsonarrayappend(printWithChildJsonArr,printWithChildJson);
                jsonput(lvl1PriceTablesJson,"printWithChildJsonArr",printWithChildJsonArr);
            }
        }
        //#################### Handling No Update Option Classes #######################
        
        //parentOptionClassJson = jsoncopy(tempParentOptionClassJson);
        //jsonput(parentOptionClassJson,"parentSKUABOCode",jsonget(lineDetailsJson,"oRCL_ABO_ActionCode_l","string","")); //Commented : 02 June 2025 : Using custom action code attribute
        jsonput(parentOptionClassJson,"parentSKUABOCode",jsonget(lineDetailsJson,"customActionCode_l_c","string","")); 
   	parentOptionClassJson = util.util_mergeJsons(parentOptionClassJson, tempParentOptionClassJson);
   	
        if(jsonget(lineDetailsJson,"_line_bom_level","integer") == 1){    
            //Added : Bibin Sajeevan : 24 February 2026 : Custom SKU Config
            if(NOT isnull(jsonget(custConfigDataJson,jsonget(lvl1PriceTablesJson,"tableHeader","string","")))){
                customSKUsDataJson = jsonget(custConfigDataJson,jsonget(lvl1PriceTablesJson,"tableHeader","string",""),"json",json());
                inputJson = json();
                jsonput(inputJson,"modelJson",modelJson);
                jsonput(inputJson,"lvl1PriceTablesJson",lvl1PriceTablesJson);
                jsonput(inputJson,"customSKUsDataJson",customSKUsDataJson);
                jsonput(inputJson,"tierIndentationJson",tierIndentationJson);
                reponse = commerce.mergeCustomSKUDetails_c(inputJson);
                jsonremove(custConfigDataJson,jsonget(lvl1PriceTablesJson,"tableHeader","string",""));
                lvl1PriceTablesJson = jsonget(reponse,"lvl1PriceTablesJson","json",lvl1PriceTablesJson);
                modelJson = jsonget(reponse,"modelJson","json",modelJson);
            }
            //Added : Bibin Sajeevan : 24 February 2026 : Custom SKU Config
            
            inputJson = json();
            jsonput(inputJson,"lvl1PriceTablesJson",lvl1PriceTablesJson);
            jsonput(inputJson,"lineDetailsJson",lineDetailsJson);
            jsonput(inputJson,"transactionPriceTableJson",transactionPriceTableJson);
            jsonput(inputJson,"modelJson",modelJson);
            if(NOT isnull(jsonget(abstractModelsJson,jsonget(modelJson,"modelVarName","string","")))){
                jsonput(inputJson,"abstractModel",jsonget(abstractModelsJson,jsonget(modelJson,"modelVarName","string","")));
                jsonput(inputJson,"pseudoModelJson",pseudoModelJson);
            } 
            reponse = commerce.collectLvl1OptionClassDetails_c(inputJson);
            lvl1PriceTablesJson = jsonget(reponse,"lvl1PriceTablesJson","json",lvl1PriceTablesJson);
            modelJson = jsonget(reponse,"modelJson","json",modelJson);
            pseudoModelJson = jsonget(reponse,"pseudoModelJson","json",pseudoModelJson);
            transactionPriceTableJson = jsonget(reponse,"transactionPriceTableJson","json",transactionPriceTableJson);
    
            sectionId = jsonget(lvl1PriceTablesJson,"SectionId","string","");
            currentOptionClass = jsonget(lvl1PriceTablesJson,"currentOptionClass","string","");
            displayStdSKU = jsonget(lvl1PriceTablesJson,"displayStdSKU","boolean",false);
            lvl1PriceTablesJson = json();
            jsonput(lvl1PriceTablesJson,"SectionId",sectionId);
            jsonput(lvl1PriceTablesJson,"currentOptionClass",currentOptionClass);
            jsonput(lvl1PriceTablesJson,"displayStdSKU",displayStdSKU);
            jsonput(lvl1PriceTablesJson,"thirdPartyTable",false);
            jsonput(lvl1PriceTablesJson,"feeThirdPartyTable",false);
            jsonput(lvl1PriceTablesJson,"lvl1OpClassDocNum",jsonget(lineDetailsJson,"_document_number"));
            jsonput(lvl1PriceTablesJson,"tableHeader",replace(replace(replace(jsonget(lineDetailsJson,"_part_desc"),"&","&amp;"),"<","&lt;"),">","&gt;"));
            //Added : 23 February 2026 : Bibin Sajeevan : Display In Transaction Summary 
            if(jsonget(proposalDataJson,"DisplayType","string","") == "DITS"){
                jsonput(lvl1PriceTablesJson,"DITS",true);
            }
            //Added : 23 February 2026 : Bibin Sajeevan : Display In Transaction Summary
        }
        
        
        //######### Skipping NO UPDATE Lines #############
        if(jsonarraysize(childSKUsJsonArray) == 0 AND jsonarraysize(jsonget(currentPriceDataJson,"AggregateJsonArray","jsonarray",jsonarray())) < 1){
            continue;
        }
        //######### Skipping NO UPDATE Lines #############
        
        //############### QRT Table Headers ###############################
        if(jsonget(proposalDataJson,"DisplayFormat","string","") == "QRT"){
            qrtHeaderStr = stringbuilder();
            altSkuDisplayName = jsonget(proposalDataJson,"AltSKUDisplay","string","");
            altSkuDisplayName = jsonget(qrtDisplayNameJson,partNumber,"string",altSkuDisplayName);
            altSkuDisplayName = replace(replace(replace(altSkuDisplayName,"&","&amp;"),"<","&lt;"),">","&gt;");
            if(altSkuDisplayName == ""){
                altSkuDisplayName = nonIndentOptionClassSKUDisplayName;
                
            }
            if(altSkuDisplayName <> ""){
                if(feeTableIndentationStr <> ""){
                    altSkuDisplayName = feeTableIndentationStr+altSkuDisplayName;
                }
                if(NOT isnull(jsonget(lvl1PriceTablesJson,"qrtOptionClassHeader"))){
                    sbappend(qrtHeaderStr,jsonget(lvl1PriceTablesJson,"qrtOptionClassHeader","string",""));
                    sbappend(qrtHeaderStr,"!_!",altSkuDisplayName);
                }
                else{
                    sbappend(qrtHeaderStr,"@_@",altSkuDisplayName);
                }
                jsonput(lvl1PriceTablesJson,"qrtOptionClassHeader",sbtostring(qrtHeaderStr));
            }
        }
        //############### QRT Table Headers ###############################
        
        //######### Skipping items without SKU ###########
        if(skuDisplayName == "" AND optionClassSKUDisplayName == "" AND NOT jsonget(lvl1PriceTablesJson,"displayStdSKU","boolean",false)){
            continue;
        }
        //######### Skipping items without SKU ###########
        
        //################ Skipping aggregated Products from Pricing Table ###################
        if(NOT isnull(jsonget(lvl1PriceTablesJson,"aggregatedProductsSKU"))){
            if(jsonpathcheck(lvl1PriceTablesJson,"$.aggregatedProductsSKU[?(@ == '"+partNumber+"')]")){
                continue;
            }
        }
        //################ Skipping aggregated Products from Pricing Table ###################        
        
        //################ Calculating Indentation ############
        jsonput(lvl1PriceTablesJson,"intendationStr",indentationStr);
        //################ Calculating Indentation ############

        //################ Setting up fee and implementation prices and deciding the pricing table ############
        totalLicensePriceRollup = jsonget(currentPriceDataJson,"totalLicensePriceRollup");
        totalAnnualfeePriceRollup = jsonget(currentPriceDataJson,"totalAnnualfeePriceRollup");
        totalYearsPriceRollup = jsonget(currentPriceDataJson,"totalYearsPriceRollup");
        totalYearsfeeNetPriceRollup = jsonget(currentPriceDataJson,"totalYearsfeeNetPriceRollup");
        totalImplementationPriceRollup = jsonget(currentPriceDataJson,"totalImplementationPriceRollup");
        totalOtherfeeNetPriceRollup = jsonget(currentPriceDataJson,"totalOtherfeeNetPriceRollup");
        totalRebateNetAmountRollup = jsonget(currentPriceDataJson,"totalRebateNetAmountRollup");
        totalRebatefeeNetPriceRollup = jsonget(currentPriceDataJson,"totalRebatefeeNetPriceRollup");
        //totalFeeTypeSKUPriceRollup = jsonget(currentPriceDataJson,"totalFeeTypeSKUPriceRollup");
        perOccurrenceFeeTypeSKUPriceRollup = jsonget(currentPriceDataJson,"perOccurrenceFeeTypeSKUPriceRollup");
        aggregateNetAmount = jsonget(currentPriceDataJson,"aggregateNetAmount");
        
        if(jsonget(proposalDataJson,"PriceTable","string","") == "unitImplTable"){
            if(NOT isnull(totalOtherfeeNetPriceRollup) OR NOT isnull(totalImplementationPriceRollup) OR NOT isnull(totalRebateNetAmountRollup) OR NOT isnull(aggregateNetAmount) OR jsonget(lvl1PriceTablesJson,"displayStdSKU","boolean",false)){
                implementation = jsonget(currentPriceDataJson,"totalImplementationPriceRollup","string","n/a");
                fee = "n/a";
                if(isnumber(implementation) AND jsonarraysize(implementationSKU) > 0){
                    implSKUNumber = jsonarrayget(implementationSKU,0,"string");
                    fee = jsonpathgetsingle(childSKUsJson,"$..[?(@._part_number == '"+implSKUNumber+"')].netPrice_l","string","0");
                    if(NOT isnull(aggregateNetAmount)){
                        fee = implementation;
                    }
                }
                jsonput(rowPricesJson,"fee",fee);
                jsonput(rowPricesJson,"implementation",implementation);
                jsonput(rowPricesJson,"currentTable","unitImplTable");
            }
        }
        elif(jsonget(proposalDataJson,"PriceTable","string","") == "perOccurrenceTable"){
            if(NOT isnull(totalOtherfeeNetPriceRollup) OR NOT isnull(perOccurrenceFeeTypeSKUPriceRollup) OR NOT isnull(aggregateNetAmount) OR jsonget(lvl1PriceTablesJson,"displayStdSKU","boolean",false)){
                implementation = jsonget(currentPriceDataJson,"perOccurrenceFeeTypeSKUPriceRollup","string","n/a");
                fee = "n/a";
                if(isnumber(implementation) AND jsonarraysize(perOccurrenceSKU) > 0){
                    perOccurenceSKUNumber = jsonarrayget(perOccurrenceSKU,0,"string");
                    fee = jsonpathgetsingle(childSKUsJson,"$..[?(@._part_number == '"+perOccurenceSKUNumber+"')].netPrice_l","string","0");
                    if(NOT isnull(aggregateNetAmount)){
                        fee = implementation;
                    }
                }
                jsonput(rowPricesJson,"fee",fee);
                jsonput(rowPricesJson,"implementation",implementation);
                jsonput(rowPricesJson,"currentTable","perOccurrenceTable");
            }
        }
        elif(jsonget(proposalDataJson,"PriceTable","string","") == "annualFeeTable"){
            if(NOT isnull(totalOtherfeeNetPriceRollup) OR NOT isnull(totalImplementationPriceRollup) OR NOT isnull(totalAnnualfeePriceRollup) OR NOT isnull(aggregateNetAmount) OR jsonget(lvl1PriceTablesJson,"displayStdSKU","boolean",false)){
                jsonput(rowPricesJson,"fee",jsonget(currentPriceDataJson,"totalAnnualfeePriceRollup","string","n/a"));
                jsonput(rowPricesJson,"implementation",jsonget(currentPriceDataJson,"totalImplementationPriceRollup","string","n/a"));
                jsonput(rowPricesJson,"currentTable","annualFeeTable");
            }
        }
        //Added : Sreekanth Vaddiboyina : 23 March 2026
        elif(jsonget(proposalDataJson,"PriceTable","string","") == "annualUnitImplTable"){
            if(NOT isnull(totalYearsPriceRollup) OR NOT isnull(totalImplementationPriceRollup) OR NOT isnull(aggregateNetAmount) OR jsonget(lvl1PriceTablesJson,"displayStdSKU","boolean",false)){
            	implementation = jsonget(currentPriceDataJson,"totalImplementationPriceRollup","string","n/a");
                fee = "n/a";
                if(isnumber(implementation) AND jsonarraysize(implementationSKU) > 0){
                    implSKUNumber = jsonarrayget(implementationSKU,0,"string");
                    fee = jsonpathgetsingle(childSKUsJson,"$..[?(@._part_number == '"+implSKUNumber+"')].netPrice_l","string","0");
                    if(NOT isnull(aggregateNetAmount)){
                        fee = implementation;
                    }
                }
                jsonput(rowPricesJson,"fee",fee);
                jsonput(rowPricesJson,"implementation",jsonget(currentPriceDataJson,"totalImplementationPriceRollup","string","n/a"));
                jsonput(rowPricesJson,"currentTable","annualUnitImplTable");
            }
        }
        //Added : Varshitha Neelapu : 3 February 2026 : AnnualTotal
        elif(jsonget(proposalDataJson,"PriceTable","string","") == "annualLicenseTable"){
            if(NOT isnull(totalYearsPriceRollup) OR NOT isnull(totalImplementationPriceRollup) OR NOT isnull(totalLicensePriceRollup) OR NOT isnull(aggregateNetAmount) OR NOT isnull(totalRebateNetAmountRollup) OR jsonget(lvl1PriceTablesJson,"displayStdSKU","boolean",false)){
                jsonput(rowPricesJson,"fee",jsonget(currentPriceDataJson,"totalLicensePriceRollup","string","n/a"));
                jsonput(rowPricesJson,"implementation",jsonget(currentPriceDataJson,"totalImplementationPriceRollup","string","n/a"));
                jsonput(rowPricesJson,"currentTable","annualLicenseTable");
            }
        }
        //Added by Varshitha Neelapu : 9 Feb 2026 for Billing frequency - Total 
        elif(jsonget(proposalDataJson,"PriceTable","string","") == "totalFeeTable"){
            if(NOT isnull(perOccurrenceFeeTypeSKUPriceRollup) OR NOT isnull(totalImplementationPriceRollup)){
                implementation = jsonget(currentPriceDataJson,"totalImplementationPriceRollup","string","n/a");
                fee = "n/a";
                unitFee = "n/a";
                perOccurrence = jsonget(currentPriceDataJson,"perOccurrenceFeeTypeSKUPriceRollup","string","0.0");
                feeImplTotal = 0;
                fee = jsonget(rowPricesJson,"fee","string","n/a");
                listPrice = jsonget(jsonget(parentOptionClassJson,"perOccurrenceSKULineDataJson","json",json()),"listPrice_l","string","Quote");
                if(isnumber(perOccurrence) AND jsonarraysize(perOccurrenceSKU) > 0){
                    perOccurrenceSKUNumber = jsonarrayget(perOccurrenceSKU,0,"string");
                    unitfee = jsonpathgetsingle(childSKUsJson,"$..[?(@._part_number == '"+perOccurrenceSKUNumber+"')].netPrice_l","string","0");
                    if(NOT isnull(aggregateNetAmount)){
                        unitFee = listPrice;
                    }
                }
                
                if(isnumber(implementation) AND jsonarraysize(implementationSKU) > 0){
                    implSKUNumber = jsonarrayget(implementationSKU,0,"string");
                    fee = jsonpathgetsingle(childSKUsJson,"$..[?(@._part_number == '"+implSKUNumber+"')].netPrice_l","string","0");
                    if(NOT isnull(aggregateNetAmount)){
                        fee = implementation;
                    }
                }
                jsonput(rowPricesJson,"unitFee",unitFee);
                jsonput(rowPricesJson,"fee",fee);
                jsonput(rowPricesJson,"implementation",implementation);
                jsonput(rowPricesJson,"currentTable","totalFeeTable");
            }
        }
        else{
            if(NOT isnull(totalOtherfeeNetPriceRollup) OR NOT isnull(totalImplementationPriceRollup) OR NOT isnull(totalLicensePriceRollup) OR NOT isnull(aggregateNetAmount) OR jsonget(lvl1PriceTablesJson,"displayStdSKU","boolean",false)){
                jsonput(rowPricesJson,"fee",jsonget(currentPriceDataJson,"totalLicensePriceRollup","string","n/a"));
                jsonput(rowPricesJson,"implementation",jsonget(currentPriceDataJson,"totalImplementationPriceRollup","string","n/a"));
                jsonput(rowPricesJson,"currentTable","licenseFeeTable");
            }
            
        }
        
        currentTable = jsonget(rowPricesJson,"currentTable");

        if(jsonget(rowPricesJson,"fee","string","n/a") == "0.0"){
            if(NOT jsonget(lvl1PriceTablesJson,"displayStdSKU","boolean",false)){
                licenseJson = jsonpathgetsingle(childSKUsJson,"$.arrayPacket[?(@.feeType_l_c== 'License' && @.rollupSKU_l_c == '"+partNumber+"')]","json",json());
                displayListPrice = jsonget(licenseJson,"displayListPrice_l_c","string","Quote");
                displayNetPrice = jsonget(licenseJson,"displayNetPrice_l_c","string","Quote");
                if(find(displayNetPrice,"$",0,1) == -1){
                    jsonput(rowPricesJson,"fee",displayNetPrice);
                }
            }
        }
        if(jsonget(rowPricesJson,"implementation","string","n/a") == "0.0"){
            if(NOT jsonget(lvl1PriceTablesJson,"displayStdSKU","boolean",false)){
                implementationJson = jsonpathgetsingle(childSKUsJson,"$.arrayPacket[?((@.feeType_l_c== 'Implementation' || @.feeType_l_c== 'Per Occurrence') && @.rollupSKU_l_c == '"+partNumber+"')]","json",json());
                //displayListPrice = jsonget(implementationJson,"displayListPrice_l_c","string","Quote");
                displayNetPrice = jsonget(implementationJson,"displayNetPrice_l_c","string","Quote");
                listPrice = jsonget(implementationJson,"listPrice_l","string","Quote");
                if(find(displayNetPrice,"$",0,1) == -1){
                    jsonput(rowPricesJson,"implementation",displayNetPrice);
                    if(currentTable == "unitImplTable"){
                        jsonput(rowPricesJson,"fee",displayNetPrice);
                        if(displayNetPrice == "Waived" OR displayNetPrice == "Included"){                            
                            if(isnumber(listPrice)){
                                if(atof(listPrice) <> 0.0){
                                    jsonput(rowPricesJson,"fee",listPrice);                                    
                                }
                            }
                        }
                    }
                }
            }
        }
        //################ Setting up fee and implementation prices and deciding the pricing table ############
        
        actualSKUNumberPriceJson = jsonget(priceDataJson,actualSKUNumber+"**"+modelDocNum,"json",json());
        aggregateJsonArray = jsonget(currentPriceDataJson,"AggregateJsonArray","jsonarray",jsonarray());
        proposalTierDisplayJsonArray =  jsonget(currentPriceDataJson,"proposalTierDisplay","jsonarray",jsonarray());
        nonBranchSKUJsonArray = jsonarray();
        if(NOT isnull(jsonget(rowPricesJson,"nonBranchSKU"))){
            nonBranchSKUDetailsJson = jsonpathgetsingle(childSKUsJson,"$..[?(@._document_number== '"+jsonget(rowPricesJson,"nonBranchSKU") +"')]","json",json());
            jsonput(rowPricesJson,"nonBranchSKUDetailsJson",nonBranchSKUDetailsJson);
            aboJson = jsonget(nonBranchSKUDetailsJson,"aBOJson_l_c","json",json());
            nonBranchSKUJsonArray = jsonpathgetsingle(aboJson,"$.currentData.proposalTierDisplay_l_c","jsonarray",jsonarray());
        }
        
        //######### Handling Miscellaneous Fees Option Classes###########
        if(jsonget(proposalDataJson,"DisplayFormat","string","") == "Miscellaneous Fees"){
            /*complSKUArr = split(jsonget(proposalDataJson,"ComplSKU","string",""),",");
            complSKUJson = jsonget(modelJson,"complSKU","json",json());
            complIndices = range(sizeofarray(complSKUArr));
            for index in complIndices{
                if(complSKUArr[index] <> ""){
                    jsonput(complSKUJson,complSKUArr[index],partNumber);
                }
            }
            if(NOT isempty(complSKUArr)){
                jsonput(modelJson,"complSKU",complSKUJson);
            }
            if(NOT isnull(jsonget(complSKUJson,partNumber))){
                continue;
            }*/
            inputJson = json();
            jsonput(inputJson,"proposalDataJson",proposalDataJson);
            jsonput(inputJson,"modelJson",modelJson);
            jsonput(inputJson,"partNumber",partNumber);
            jsonput(inputJson,"function","complSKU");
            response = util.util_checkComplementary(inputJson);
            modelJson = jsonget(response,"modelJson","json",modelJson);
            if(jsonget(response,"isCompl","boolean",false)){
                continue;
            } 
            miscellaneousHeaderStr = stringbuilder();
            //Added : Bibin Sajeevan : 02 January 2026 : Multiple consecutive headers in miscellenous table (9939-9941-9424)
            sbappend(miscellaneousHeaderStr,jsonget(lvl1PriceTablesJson,"miscellaneousHeader","string",""));
            //Added : Bibin Sajeevan : 02 January 2026 : Multiple consecutive headers in miscellenous table
            sbappend(miscellaneousHeaderStr,"@_@",skuDisplayName);
            jsonput(lvl1PriceTablesJson,"miscellaneousHeader",sbtostring(miscellaneousHeaderStr));
        }
        //######### Handling Miscellaneous Fees Option Classes###########
        
        //####################### Non Price Row ########################
        elif(optionClassSKUDisplayName <> ""){
            currentTable = jsonget(proposalDataJson,"PriceTable","string","");
            
            if(jsonget(lvl1PriceTablesJson,"hasAPriorHeader","boolean",false)){
                sbappend(priceTableRowStr,"!_!",optionClassSKUDisplayName);
                jsonremove(lvl1PriceTablesJson,"hasAPriorHeader");
            }
            else{
                sbappend(priceTableRowStr,rowSecSep,optionClassSKUDisplayName);
            }
            jsonput(lvl1PriceTablesJson,"hasAPriorHeader",true);
            if(jsonget(lvl1PriceTablesJson,"displayStdSKU","boolean",false)){
                jsonput(rowPricesJson,"displayStdSKUHeader",sbtostring(priceTableRowStr));
                jsonput(parentOptionClassJson,"hasDisplayStdSKUHeader",true);
            }
            //jsonput(lvl1PriceTablesJson,"lastOptionClassEndIndex",currentTableStrSize+3+len(optionClassSKUDisplayName));
            if(rowSecSep == "@_@"){
                rowSecSep = "!_!";
            }
            //--------- Clubbing the headers with next line in price table --------------------
        }
        elif(jsonget(lvl1PriceTablesJson,"displayStdSKU","boolean",false)){
            jsonput(lvl1PriceTablesJson,"displayStdSKUBlankHeaderRowSecSep",rowSecSep);
        }
        //####################### Non Price Row ########################
        
        //####################### Handling Aggregate Pricing Data ########################
        if(jsonarraysize(aggregateJsonArray)> 0){
            tierPricesStr = stringbuilder();
            jsonput(lvl1PriceTablesJson,"displayStdSKU",false);
            if(jsonget(lineDetailsJson,"aggregatedProducts_l_c") <> ""){
                    aggrProdSkuDisplayNamesJson = json();
                    aggregatedProductsSKUStr = replace(jsonget(currentPriceDataJson,"aggrProdSkuNumber","string",""),",","",1);
                    aggregatedProductsArr = split(aggregatedProductsSKUStr,",");
                    orderNoArr = integer[];
                    indices = range(sizeofarray(aggregatedProductsArr));
                    for index in indices{
                        aggregatedChildSKUNum = aggregatedProductsArr[index];
                        aggregatedChildSKUJson = jsonget(tableDetailsJson,aggregatedChildSKUNum,"json",json());
                        aggregatedChildSKUDetailsJson = jsonpathgetsingle(getBOMDetailsJson,"$.children[?(@.fields._part_number == '"+aggregatedChildSKUNum+"')].fields","json",json());
                        if(jsonget(aggregatedChildSKUDetailsJson,"feeType_l_c","string","") == "Monthly" OR jsonget(aggregatedChildSKUDetailsJson,"feeType_l_c","string","") == "Included" ){
                            aggregatedChildSKUDisplay = jsonget(aggregatedChildSKUJson,"SKUDisplay","string","");
                            aggregatedChildSKUDisplay = jsonget(displayNameJson,aggregatedChildSKUNum,"string",aggregatedChildSKUDisplay);
                            if(aggregatedChildSKUDisplay <> "" AND aggregatedChildSKUDisplay <> optionClassSKUDisplayName){
                                //jsonput(aggrProdSkuDisplayNamesJson,aggregatedChildSKUDisplay,"");
                                orderNo = jsonget(aggregatedChildSKUDetailsJson,"_sequence_number","integer",0);
                                append(orderNoArr,orderNo);
                                jsonput(aggrProdSkuDisplayNamesJson,string(orderNo),aggregatedChildSKUDisplay);
                            }
                        }
                    }
                    //Added : Bibin Sajeevan : 23 January 2026 : Correcting Aggregated SKUs Order (15579-15583-44061)
                    aggrProdSkuDisplayNameOrderArr = sort(orderNoArr);
                    aggrProdSkuDisplayNameArr = string[];
                    for currentOrderNo in aggrProdSkuDisplayNameOrderArr{
                        append(aggrProdSkuDisplayNameArr,jsonget(aggrProdSkuDisplayNamesJson,string(currentOrderNo),"string",""));
                    }
                    //Added : Bibin Sajeevan : 23 January 2026 : Correcting Aggregated SKUs Order
                    if(sizeofarray(aggrProdSkuDisplayNameArr) > 0){
                        //jsonput(parentOptionClassJson,"aggregatedProducts","!_!Includes the following:\\l"+replace(replace(replace(join(aggrProdSkuDisplayNameArr,", "),"&","&amp;"),"<","&lt;"),">","&gt;"));
                        jsonput(parentOptionClassJson,"inclusionHeader","!_!Includes: ");
                        //jsonput(parentOptionClassJson,"aggregatedProducts","!_!Includes: "+replace(replace(replace(join(aggrProdSkuDisplayNameArr,", "),"&","&amp;"),"<","&lt;"),">","&gt;"));
                        jsonput(parentOptionClassJson,"aggregatedProducts",replace(replace(replace(join(aggrProdSkuDisplayNameArr,", "),"&","&amp;"),"<","&lt;"),">","&gt;"));
                    }
            }
            
            inputJson = json();
            jsonput(inputJson,"aggregateJsonArray",aggregateJsonArray);
            jsonput(inputJson,"lineDetailsJson",lineDetailsJson);
            jsonput(inputJson,"currentPriceDataJson",currentPriceDataJson);
            jsonput(inputJson,"proposalDataJson",proposalDataJson);
            //jsonput(inputJson,"currentTable",currentTable);
            jsonput(inputJson,"tierIndentationJson",tierIndentationJson);
            jsonput(inputJson,"rowPricesJson",rowPricesJson);
            jsonput(inputJson,"lvl1PriceTablesJson",lvl1PriceTablesJson);

            reponse = commerce.collectAggregateDetails_c(inputJson);
            sbappend(priceTableRowStr,jsonget(reponse,"priceTableRowStr","string",""));
            jsonput(parentOptionClassJson,"tierPricesStr",jsonget(reponse,"tierPricesStr","string",""));
            rowPricesJson = jsonget(reponse,"rowPricesJson","json",json());
            lvl1PriceTablesJson = jsonget(reponse,"lvl1PriceTablesJson","json",json());
        }
        elif(NOT isnull(jsonget(currentPriceDataJson,"aggregateImplementationFee")) OR NOT isnull(jsonget(currentPriceDataJson,"aggregateLicenseFee"))){    
            priceTableRowStr = stringbuilder();
            jsonput(lvl1PriceTablesJson,"displayStdSKU",false);
            skuDisplayName = optionClassSKUDisplayName;
            if(jsonget(lineDetailsJson,"aggregatedProducts_l_c") <> ""){
                    aggrProdSkuDisplayNamesJson = json();
                    aggregatedProductsSKUStr = replace(jsonget(currentPriceDataJson,"aggrProdSkuNumber","string",""),",","",1);
                    aggregatedProductsArr = split(aggregatedProductsSKUStr,",");
                    orderNoArr = integer[];
                    indices = range(sizeofarray(aggregatedProductsArr));
                    for index in indices{
                        aggregatedChildSKUNum = aggregatedProductsArr[index];
                        aggregatedChildSKUJson = jsonget(tableDetailsJson,aggregatedChildSKUNum,"json",json());
                        aggregatedChildSKUDetailsJson = jsonpathgetsingle(getBOMDetailsJson,"$.children[?(@.fields._part_number == '"+aggregatedChildSKUNum+"')].fields","json",json());
                        aggregatedChildSKUDisplay = jsonget(aggregatedChildSKUJson,"SKUDisplay","string","");
                        aggregatedChildSKUDisplay = jsonget(displayNameJson,aggregatedChildSKUNum,"string",aggregatedChildSKUDisplay);
                        if(aggregatedChildSKUDisplay <> "" AND aggregatedChildSKUDisplay <> skuDisplayName){
                            orderNo = jsonget(aggregatedChildSKUDetailsJson,"_sequence_number","integer",0);
                            append(orderNoArr,orderNo);
                            jsonput(aggrProdSkuDisplayNamesJson,string(orderNo),aggregatedChildSKUDisplay);
                        }
                    }
                    //Added : Bibin Sajeevan : 23 January 2026 : Correcting Aggregated SKUs Order
                    aggrProdSkuDisplayNameOrderArr = sort(orderNoArr);
                    aggrProdSkuDisplayNameArr = string[];
                    for currentOrderNo in aggrProdSkuDisplayNameOrderArr{
                        append(aggrProdSkuDisplayNameArr,jsonget(aggrProdSkuDisplayNamesJson,string(currentOrderNo),"string",""));
                    }
                    //Added : Bibin Sajeevan : 23 January 2026 : Correcting Aggregated SKUs Order
                    if(sizeofarray(aggrProdSkuDisplayNameArr) > 0){
                        //jsonput(parentOptionClassJson,"aggregatedProducts","!_!Includes the following:\\l"+replace(replace(replace(join(aggrProdSkuDisplayNameArr,", "),"&","&amp;"),"<","&lt;"),">","&gt;"));
                        jsonput(parentOptionClassJson,"inclusionHeader","!_!Includes: ");
                        //jsonput(parentOptionClassJson,"aggregatedProducts","!_!Includes: "+replace(replace(replace(join(aggrProdSkuDisplayNameArr,", "),"&","&amp;"),"<","&lt;"),">","&gt;"));
                        jsonput(parentOptionClassJson,"aggregatedProducts",replace(replace(replace(join(aggrProdSkuDisplayNameArr,", "),"&","&amp;"),"<","&lt;"),">","&gt;"));
                    }
            }
            license = jsonget(currentPriceDataJson,"aggregateLicenseFee","string","n/a");
            implmentation = jsonget(currentPriceDataJson,"aggregateImplementationFee","string","n/a");
            if(implmentation <> "n/a"){
                if(jsonget(proposalDataJson,"PriceTable","string","") == "unitImplTable"){
                    license = implmentation;
                }
            }
            if(NOT isnull(jsonget(currentPriceDataJson,"aggregateImplementationFee"))){
                if(jsonget(currentPriceDataJson,"aggregateImplementationFee","float",0.0) == 0.0){
                    implmentation = "Waived";
                    if(currentTable == "unitImplTable"){
                        license = jsonget(currentPriceDataJson,"aggregateImplementationListAmount","string","0.0");
                    }
                    //Jira 2114 Issue 3
                    if(jsonget(currentPriceDataJson,"aggregateImplementationFeeLabel","string","") == "Quote"){
                        implmentation = "Quote";
                        license = "Quote";
                    }
                    //Jira 2114 Issue 3
                }
            }
            if(NOT isnull(jsonget(currentPriceDataJson,"aggregateLicenseFee"))){
                if(jsonget(currentPriceDataJson,"aggregateLicenseFee","float",0.0) == 0.0){
                    license = "Waived";
                }
            }
            sbappend(priceTableRowStr,rowSecSep,skuDisplayName,"*_*","1","*_*","n/a","*_*","n/a","*_*",license,"*_*",implmentation);
            jsonput(rowPricesJson,"monthlyTotal",jsonget(currentPriceDataJson,"aggregateNetAmount","float",0.0));
            jsonput(rowPricesJson,"listTotal",jsonget(currentPriceDataJson,"aggregateListAmount","float",0.0));
			
	    //Included below logic to consider the Annual total for Aggregation. - Chandu.P - 3/30/2026
	    jsonput(rowPricesJson,"annualTotal",jsonget(currentPriceDataJson,"aggregateAnnualNetAmount","float",0.0));
            
            jsonput(rowPricesJson,"licenseTotal",jsonget(currentPriceDataJson,"aggregateLicenseFee","float",0.0));
            jsonput(rowPricesJson,"implTotal",jsonget(currentPriceDataJson,"aggregateImplementationFee","float",0.0));
			
            //Added : Bibin Sajeevan : 13 February 2026 : Discount Assumption Prices
            jsonput(rowPricesJson, "licenseListTotal", jsonget(currentPriceDataJson, "aggregateLicenseListAmount", "float", 0.0));
            jsonput(rowPricesJson, "implListTotal", jsonget(currentPriceDataJson, "aggregateImplementationListAmount", "float", 0.0));
	    //Added : Bibin Sajeevan : 06 April 2026 : Discount Assumption Annual Price
            jsonput(rowPricesJson, "annualListTotal", jsonget(currentPriceDataJson, "aggregateAnnualListAmount", "float", 0.0));
	    //Added : Bibin Sajeevan : 06 April 2026 : Discount Assumption Annual Price
            //Added : Bibin Sajeevan : 13 February 2026 : Discount Assumption Prices
			
            if(NOT isnull(jsonget(currentPriceDataJson,"aggregatePlusMinNetAmount"))){
                jsonput(rowPricesJson,"monthlyTotal",jsonget(rowPricesJson,"monthlyTotal","float",0.0) - jsonget(currentPriceDataJson,"aggregatePlusMinNetAmount","float",0.0));
                jsonput(rowPricesJson,"listTotal",jsonget(rowPricesJson,"listTotal","float",0.0) - jsonget(currentPriceDataJson,"aggregatePlusMinListAmount","float",0.0));
            }
        
            if(NOT isnull(jsonget(currentPriceDataJson,"aggrProdSkuNumber"))){
                aggregatedProductsSKUStr = replace(jsonget(currentPriceDataJson,"aggrProdSkuNumber","string",""),",","",1);
                aggregatedProductsSKUStr = "[\""+ replace(aggregatedProductsSKUStr,",","\",\"") + "\"]";
                jsonput(lvl1PriceTablesJson,"aggregatedProductsSKU",jsonarray(aggregatedProductsSKUStr));
            }
        }
        //####################### Handling Aggregate Pricing Data ######################
        
        //####################### Handling QRT items ###################################
        elif((qrtSKU == "QRT" OR qrtSKU == "QRT1" OR qrtSKU == "QRT2" OR qrtSKU == "QRT3") AND jsonarraysize(nonBranchSKUJsonArray) == 0){
            qRTSKUJson = jsonpathgetsingle(childSKUsJson,"$.arrayPacket[?(@._part_number == '"+actualSKUNumber+"')]","json",json());
            inputJson = json();
            jsonput(inputJson,"qrtSKU",qrtSKU);
            jsonput(inputJson,"currentPriceDataJson",currentPriceDataJson);
            jsonput(inputJson,"priceTableRowStr",sbtostring(priceTableRowStr));
            jsonput(inputJson,"qRTSKUJson",qRTSKUJson);
            jsonput(inputJson,"rowSecSep",rowSecSep);
            jsonput(inputJson,"skuDisplayName",skuDisplayName);
            jsonput(inputJson,"actualSKUTableJson",actualSKUTableJson);
            jsonput(inputJson,"rowPricesJson",rowPricesJson);
            jsonput(inputJson,"lvl1PriceTablesJson",lvl1PriceTablesJson);
            response = commerce.applyQRTPricing_c(inputJson);
            priceTableRowStr = sbappend(priceTableRowStr,jsonget(response,"priceTableRowStr","string",""));
            rowPricesJson = jsonget(response,"rowPricesJson","json",rowPricesJson);
        }
        //####################### Handling QRT items #######################################
        
        //####################### Handling Proposal Tier Pricing Data ######################
        elif(jsonarraysize(proposalTierDisplayJsonArray) > 1){
            //Updated : Bibin Sajeevan : 31 March 2026 : Masking Display Name : QTCCPQ - 5443
            if(find(skuDisplayName,"[MASKED]") == -1){
            	sbappend(priceTableRowStr,rowSecSep,skuDisplayName);
            }
            //Updated : Bibin Sajeevan : 31 March 2026 : Masking Display Name : QTCCPQ - 5443
            tierSize = jsonarraysize(proposalTierDisplayJsonArray);
            //Updated : Bibin Sajeevan : 27 April 2026 : CSF Tiered License
            //if(jsonget(implementationSKUProposalDataJson,"DisplayType","string","") == "DIT"){
            if(jsonget(jsonget(parentOptionClassJson,"implementationSKUProposalDataJson","json",json()),"DisplayType","string","") == "DIT" OR jsonget(jsonget(parentOptionClassJson,"licenseSKUProposalDataJson","json",json()),"DisplayType","string","") == "DIT"){
            //Updated : Bibin Sajeevan : 27 April 2026 : CSF Tiered License
                inputJson = json();
                jsonput(inputJson,"parentOptionClassJson",parentOptionClassJson);
                jsonput(inputJson,"tierIndentationJson",tierIndentationJson);
                reponse = commerce.collectHybridTierDetails_c(inputJson);
                
            }
            else{
                if((jsonget(actualSKUNumberPriceJson,"tierDisplayType","string","") == "n")){
                    if(jsonarraysize(proposalTierDisplayJsonArray) > 0){
                        jsonarrayremove(proposalTierDisplayJsonArray,0);
                        currentTierIndexForMonthlyFee = jsonget(currentPriceDataJson,"currentTierIndexForMonthlyFee","integer",0);
                        jsonput(currentPriceDataJson,"currentTierIndexForMonthlyFee",currentTierIndexForMonthlyFee-1);
                    }
                }
                inputJson = json();
                jsonput(inputJson,"lineDetailsJson",lineDetailsJson);
                jsonput(inputJson,"currentPriceDataJson",currentPriceDataJson);
                jsonput(inputJson,"proposalTierDisplayJsonArray",proposalTierDisplayJsonArray);
                jsonput(inputJson,"nonBranchSKUJsonArray",nonBranchSKUJsonArray);
                jsonput(inputJson,"actualSKUTableJson",actualSKUTableJson);
                jsonput(inputJson,"proposalDataJson",proposalDataJson);
                jsonput(inputJson,"tierIndentationJson",tierIndentationJson);
                if(jsonget(currentPriceDataJson,"totalOtherfeeNetPriceRollup","string","") == "0.0" AND NOT jsonget(lvl1PriceTablesJson,"displayStdSKU","boolean",false)){
                    monthlySKUJson = jsonpathgetsingle(childSKUsJson,"$.arrayPacket[?(@.feeType_l_c== 'Monthly' && @.rollupSKU_l_c == '"+partNumber+"')]","json",json());
                    displayNetPrice = jsonget(monthlySKUJson,"displayNetPrice_l_c","string","");
                    listFeeMinimum = jsonget(monthlySKUJson,"listFeeMinimumDisplay_l_c","string","");
                    if(displayNetPrice == "Quote" AND listFeeMinimum == "Quote"){
                        jsonput(rowPricesJson,"quoteMinimum",true);
                    }
                }
                jsonput(inputJson,"rowPricesJson",rowPricesJson);
                reponse = commerce.collectTierDetails_c(inputJson);
            }
            sbappend(priceTableRowStr,jsonget(reponse,"priceTableRowStr","string",""));
            if(tierSize <> 1){
                jsonput(parentOptionClassJson,"tierPricesStr",jsonget(reponse,"tierPricesStr","string",""));
            }
            jsonput(rowPricesJson,"monthlyTotal",jsonget(currentPriceDataJson,"totalOtherfeeNetAmountRollup","float",0.0));
            jsonput(rowPricesJson,"licenseTotal",jsonget(currentPriceDataJson,"totalLicensePriceRollup","float",0.0));
            jsonput(rowPricesJson,"implTotal",jsonget(currentPriceDataJson,"totalImplementationPriceRollup","float",0.0));
            jsonput(rowPricesJson,"totalFee",jsonget(currentPriceDataJson,"perOccurrenceFeeTypeSKUPriceRollup","float",0.0));
	    jsonput(rowPricesJson,"annualTotal",jsonget(currentPriceDataJson,"totalYearsPriceRollup","float",0.0));
			
            //Added : Bibin Sajeevan : 13 February 2026 : Discount Assumption Prices
            jsonput(rowPricesJson, "licenseListTotal", jsonget(currentPriceDataJson, "totalLicenseListAmountRollup", "float", 0.0));
            jsonput(rowPricesJson, "implListTotal", jsonget(currentPriceDataJson, "totalImplementationListAmountRollup", "float", 0.0));
	    //Added : Bibin Sajeevan : 06 April 2026 : Discount Assumption Annual Price
            jsonput(rowPricesJson, "annualListTotal", jsonget(currentPriceDataJson, "totalYearsListAmountRollup", "float", 0.0));
	    //Added : Bibin Sajeevan : 06 April 2026 : Discount Assumption Annual Price
            //Added : Bibin Sajeevan : 13 February 2026 : Discount Assumption Prices
			
            if(NOT jsonget(rowPricesJson,"quoteMinimum","boolean",false)){
                jsonput(rowPricesJson,"listTotal",jsonget(currentPriceDataJson,"totalOtherfeeListAmountRollup","float",0.0));
            }
        }
        //####################### Handling Proposal Tier Pricing Data ######################
        
        
        //####################### Handling Normal Option Classes  ##########################
        elif(NOT isnull(jsonget(currentPriceDataJson,"totalOtherfeeNetPriceRollup"))){
            tempPriceTableRowStr= stringbuilder();
            sbappend(tempPriceTableRowStr,rowSecSep,skuDisplayName);
            unitFee = jsonget(currentPriceDataJson,"totalOtherfeeNetPriceRollup","string","0.0");
            monthly = jsonget(currentPriceDataJson,"totalOtherfeeNetAmountRollup","string","0.0");
            if(jsonget(currentPriceDataJson,"IndividualMinApplying","boolean",false)){
                if(jsonget(actualSKUTableJson,"DisplayType","string","") == "NPM"){
                    unitFee = jsonget(currentPriceDataJson,"totalOtherfeeNetAmountRollup","string","0.0");
                }
                else{
                    sbappend(tempPriceTableRowStr," - Minimum Fee");
                }
            }
            fee = jsonget(rowPricesJson,"fee","string","n/a");
            implementation = jsonget(rowPricesJson,"implementation","string","n/a");
            jsonput(rowPricesJson,"monthlyTotal",atof(monthly));
            monthlySKUJson = jsonpathgetsingle(childSKUsJson,"$.arrayPacket[?(@._part_number == '"+actualSKUNumber+"')]","json",json());
            if(unitFee == "0.0" AND NOT jsonget(lvl1PriceTablesJson,"displayStdSKU","boolean",false)){
                //displayListPrice = jsonget(monthlySKUJson,"displayListPrice_l_c","string","");
                displayNetPrice = jsonget(monthlySKUJson,"displayNetPrice_l_c","string","");
                listFeeMinimum = jsonget(monthlySKUJson,"listFeeMinimumDisplay_l_c","string","");
                listPrice = jsonget(monthlySKUJson,"listPrice_l","string","");
                if(find(displayNetPrice,"$",0,1) == -1){
                    unitFee = displayNetPrice;
                    monthly = displayNetPrice;
                    if(displayNetPrice == "Quote" AND listFeeMinimum == "Quote"){
                        //Updated : Bibin Sajeevan : 20 January 2026 : Not setting list price in Unit Fee when list price is quote (976-14376-56622)
                        displayListPrice = jsonget(monthlySKUJson,"displayListPrice_l_c","string","");
                        if(atof(listPrice) <> 0 AND displayListPrice <> "Quote"){
                            unitFee = listPrice;
                        }
                        //Updated : Bibin Sajeevan : 20 January 2026 : Not setting list price in Unit Fee when list price is quote
                        jsonput(rowPricesJson,"quoteMinimum",true);
                    }    
                }
                if(displayNetPrice == "Waived" OR displayNetPrice == "Included"){                    
                    if(isnumber(listPrice)){
                        if(atof(listPrice) <> 0.0){                            
                            unitFee = listPrice;
                        }
                    }
                }
            } 
            jsonput(rowPricesJson,"monthlyTotal",jsonget(currentPriceDataJson,"totalOtherfeeNetAmountRollup","float",0.0));
            jsonput(rowPricesJson,"licenseTotal",jsonget(currentPriceDataJson,"totalLicensePriceRollup","float",0.0));
            jsonput(rowPricesJson,"implTotal",jsonget(currentPriceDataJson,"totalImplementationPriceRollup","float",0.0));
            //Added : 13 February 2026 : Discount Assumption Prices
            jsonput(rowPricesJson,"licenseListTotal",jsonget(currentPriceDataJson,"totalLicenseListAmountRollup","float",0.0));
            jsonput(rowPricesJson,"implListTotal",jsonget(currentPriceDataJson,"totalImplementationListAmountRollup","float",0.0));
            //Added : 13 February 2026 : Discount Assumption Prices
            if(NOT jsonget(rowPricesJson,"quoteMinimum","boolean",false)){
                jsonput(rowPricesJson,"listTotal",jsonget(currentPriceDataJson,"totalOtherfeeListAmountRollup","float",0.0));
            }
            quantityStr = jsonget(currentPriceDataJson,"_price_quantity","string","1");
            //Updated : Bibin Sajeevan : 13 September 2025 : FCM New Pattern
            if(jsonget(monthlySKUJson,"aCVExclusionFlag_l_c","boolean",false)){
                quantityStr = "0";
                monthly = "0.0";
            }
            //Updated : Bibin Sajeevan : 13 September 2025 : FCM New Pattern
	    //Added : Bibin Sajeevan : 04 March 2026 : Price Qty Zero in Proposal 
	    if(jsonget(jsonget(parentOptionClassJson,"monthlySKUProposalDataJson","json",json()),"DisplayType","string","") == "PQZ"){
	       quantityStr = "0";
	       monthly = "0.0";
	    }
            //Added : Bibin Sajeevan : 04 March 2026 : Price Qty Zero in Proposal
            if(quantityStr == "0"){
                quantityStr = "-";
            }
            //Added : Bibin Sajeevan : 01 April 2026 : Display as Tier : QTCCPQ - 5450
            if(jsonget(jsonget(parentOptionClassJson,"monthlySKUProposalDataJson","json",json()),"DisplayType","string","") == "DAT"){
                sbappend(priceTableRowStr,sbtostring(tempPriceTableRowStr));
                tierStr = stringbuilder();
                startIndent = jsonget(tierIndentationJson,string(len("0")));
	       	sbappend(tierStr,"!_!",startIndent, "0 - 1","*_*",quantityStr,"*_*",unitFee,"*_*",monthly,"*_*",fee,"*_*",implementation);
                jsonput(parentOptionClassJson,"tierPricesStr",sbtostring(tierStr));
	    }
	    //Added : Bibin Sajeevan : 01 April 2026 : Display as Tier : QTCCPQ - 5450
            elif((jsonget(actualSKUNumberPriceJson,"tierDisplayType","string","") <> "n")){
                sbappend(priceTableRowStr,sbtostring(tempPriceTableRowStr),"*_*",quantityStr,"*_*",unitFee,"*_*",monthly,"*_*",fee,"*_*",implementation);
            }
        }
        //Added : Varshitha Neelapu : 3 February 2026 : AnnualTotal
        elif(NOT isnull(jsonget(currentPriceDataJson,"totalYearsPriceRollup"))){ 
            rowStr = stringbuilder();
            unitFee = "n/a";
            annual = "n/a";
            feeImplTotal = 0;
            fee = jsonget(rowPricesJson,"fee","string","n/a");
            unitFee = jsonget(currentPriceDataJson,"totalYearsfeeNetPriceRollup","string","0.0");
            license = jsonget(currentPriceDataJson,"totalLicensePriceRollup","string","n/a");
            annual = jsonget(currentPriceDataJson,"totalYearsPriceRollup","string","0.0");
            implementation = jsonget(rowPricesJson,"implementation","string","n/a");    
            if(jsonget(currentPriceDataJson,"IndividualMinApplying","boolean",false)){
                if(jsonget(actualSKUTableJson,"DisplayType","string","") == "NPM"){
                    if(jsonget(rowPricesJson,"currentTable","string","") == "annualLicenseTable"){
                        annual = jsonget(currentPriceDataJson,"totalYearsPriceRollup","string","0.0");
                    }
                }
            }
            //Added : Varshitha Neelapu : 6 February 2026 : Annual Quote 
            annualSKUJson = jsonpathgetsingle(childSKUsJson,"$.arrayPacket[?(@.feeType_l_c == 'Years' && @._part_number == '"+actualSKUNumber+"')]","json",json());
            //annualSKUJson = jsonpathgetsingle(childSKUsJson,"$.arrayPacket[?(@.feeType_l_c == 'Years' && @.rollupSKU_l_c == '"+partNumber+"')]","json",json()); 
            if(unitFee == "0.0" AND NOT jsonget(lvl1PriceTablesJson,"displayStdSKU","boolean",false) AND NOT isnull(jsonget(annualSKUJson,"_part_number"))){
                displayNetPrice = jsonget(annualSKUJson,"displayNetPrice_l_c","string","");
                //print jsontostr(annualSKUJson);
                listFeeMinimum = jsonget(annualSKUJson,"listFeeMinimumDisplay_l_c","string","");
                listPrice = jsonget(annualSKUJson,"listPrice_l","string","");
                if(find(displayNetPrice,"$",0,1) == -1){
                    unitFee = displayNetPrice;
                    annual = displayNetPrice;
                    if(displayNetPrice == "Quote" AND listFeeMinimum == "Quote"){
                        displayListPrice = jsonget(annualSKUJson,"displayListPrice_l_c","string","");
                        if(atof(listPrice) <> 0 AND displayListPrice <> "Quote"){
                            unitFee = listPrice;
                        }
                        jsonput(rowPricesJson,"quoteMinimum",true);
                    }    
                }
            }
            jsonput(rowPricesJson,"annualTotal",jsonget(currentPriceDataJson,"totalYearsPriceRollup","float",0.0));
            jsonput(rowPricesJson,"licenseTotal",jsonget(currentPriceDataJson,"totalLicensePriceRollup","float",0.0));
            jsonput(rowPricesJson,"implTotal",jsonget(currentPriceDataJson,"totalImplementationPriceRollup","float",0.0));
            //Added : 13 February 2026 : Discount Assumption Prices
            jsonput(rowPricesJson,"licenseListTotal",jsonget(currentPriceDataJson,"totalLicenseListAmountRollup","float",0.0));
            jsonput(rowPricesJson,"implListTotal",jsonget(currentPriceDataJson,"totalImplementationListAmountRollup","float",0.0));
	    //Added : Bibin Sajeevan : 06 April 2026 : Discount Assumption Annual Price
            jsonput(rowPricesJson, "annualListTotal", jsonget(currentPriceDataJson, "totalYearsListAmountRollup", "float", 0.0));
	    //Added : Bibin Sajeevan : 06 April 2026 : Discount Assumption Annual Price
            //Added : 13 February 2026 : Discount Assumption Prices
            quantityStr = jsonget(currentPriceDataJson,"_price_quantity","string","1");
            if(quantityStr == "0"){
                quantityStr = "-";
            }
            sbappend(rowStr,rowSecSep,skuDisplayName,"*_*",quantityStr,"*_*",unitFee,"*_*",annual,"*_*",fee,"*_*",implementation);
                        
            if((jsonget(actualSKUNumberPriceJson,"tierDisplayType","string","") <> "n")){
                sbappend(priceTableRowStr,sbtostring(rowStr));
            }
        }
        //Added by Varshitha Neelapu : 9 Feb 2026 for Billing frequency - Total 
        elif(NOT isnull(jsonget(currentPriceDataJson,"perOccurrenceFeeTypeSKUPriceRollup"))){ //OR NOT isnull(jsonget(currentPriceDataJson,"totalImplementationPriceRollup"))){
            rowStr = stringbuilder();
            unitFee = jsonget(rowPricesJson,"unitFee","string","n/a");
            perOccurrence = jsonget(currentPriceDataJson,"perOccurrenceFeeTypeSKUPriceRollup","string","0.0");
            feeImplTotal = 0;
            fee = jsonget(rowPricesJson,"fee","string","n/a");
            implementation = jsonget(rowPricesJson,"implementation","string","n/a");
            if(jsonget(currentPriceDataJson,"IndividualMinApplying","boolean",false)){
                if(jsonget(actualSKUTableJson,"DisplayType","string","") == "NPM"){
                    if(jsonget(rowPricesJson,"currentTable","string","") == "totalFeeTable"){
                        fee = jsonget(currentPriceDataJson,"totalImplementationPriceRollup","string","0.0");
                    }
                }
            }
            jsonput(rowPricesJson,"totalFee",jsonget(currentPriceDataJson,"perOccurrenceFeeTypeSKUPriceRollup","float",0.0));
            jsonput(rowPricesJson,"implTotal",jsonget(currentPriceDataJson,"totalImplementationPriceRollup","float",0.0));
            //Added : 13 February 2026 : Discount Assumption Prices
            jsonput(rowPricesJson,"implListTotal",jsonget(currentPriceDataJson,"totalImplementationListAmountRollup","float",0.0));
            //Added : 13 February 2026 : Discount Assumption Prices
            quantityStr = jsonget(currentPriceDataJson,"_price_quantity","string","1");
            if(quantityStr == "0"){
                quantityStr = "-";
            }
            if(jsonget(proposalDataJson,"PriceTable","string","") == "totalFeeTable"){
            	sbappend(rowStr,rowSecSep,skuDisplayName,"*_*",quantityStr,"*_*",unitFee,"*_*",perOccurrence,"*_*",fee,"*_*",implementation);
            }
            else{
            	unit = jsonget(currentPriceDataJson,"uOM","string","");
                if(jsonget(actualSKUTableJson,"FeeTableUnit","string","") <> ""){
                    unit = jsonget(actualSKUTableJson,"FeeTableUnit","string","");
                }
                sbappend(rowStr,rowSecSep,skuDisplayName,"*_*",quantityStr,"*_*",unit,"*_*",fee,"*_*",implementation);
            }
            if((jsonget(actualSKUNumberPriceJson,"tierDisplayType","string","") <> "n")){
                sbappend(priceTableRowStr,sbtostring(rowStr));
            }
        }
        elif(NOT isnull(jsonget(currentPriceDataJson,"totalLicensePriceRollup")) OR NOT isnull(jsonget(currentPriceDataJson,"totalImplementationPriceRollup"))){
            rowStr = stringbuilder();
            unitFee = "n/a";
            monthly = "n/a";
            feeImplTotal = 0;
            fee = jsonget(rowPricesJson,"fee","string","n/a");
            implementation = jsonget(rowPricesJson,"implementation","string","n/a");
            // Updated : Bibin Sajeevan : 25 August 2025 : SCD - Minimum Applying on One Time
            if(jsonget(currentPriceDataJson,"IndividualMinApplying","boolean",false)){
                if(jsonget(actualSKUTableJson,"DisplayType","string","") == "NPM"){
                    if(jsonget(rowPricesJson,"currentTable","string","") == "unitImplTable" OR jsonget(rowPricesJson,"currentTable","string","") == "totalFeeTable"){
                        fee = jsonget(currentPriceDataJson,"totalImplementationPriceRollup","string","0.0");
                    }
                }
            }
            // Updated : Bibin Sajeevan : 25 August 2025 : SCD - Minimum Applying on One Time
            jsonput(rowPricesJson,"monthlyTotal",jsonget(currentPriceDataJson,"totalOtherfeeNetAmountRollup","float",0.0));
            if(jsonget(actualSKUTableJson,"DisplayType","string","") <> "SIPT"){//XBD TEST
                jsonput(rowPricesJson,"licenseTotal",jsonget(currentPriceDataJson,"totalLicensePriceRollup","float",0.0));
                jsonput(rowPricesJson,"licenseListTotal",jsonget(currentPriceDataJson,"totalLicenseListAmountRollup","float",0.0));
            }
            jsonput(rowPricesJson,"implTotal",jsonget(currentPriceDataJson,"totalImplementationPriceRollup","float",0.0));
            jsonput(rowPricesJson,"listTotal",jsonget(currentPriceDataJson,"totalOtherfeeListAmountRollup","float",0.0));
            //Added : 13 February 2026 : Discount Assumption Prices
            
            jsonput(rowPricesJson,"implListTotal",jsonget(currentPriceDataJson,"totalImplementationListAmountRollup","float",0.0));
            //Added : 13 February 2026 : Discount Assumption Prices
            quantityStr = jsonget(currentPriceDataJson,"_price_quantity","string","1");

            if(quantityStr == "0"){
                quantityStr = "-";
            }
            if(jsonget(jsonget(parentOptionClassJson,"implementationSKUProposalDataJson","json",json()),"DisplayType","string","") == "DIT"){
                sbappend(rowStr,rowSecSep,skuDisplayName);
                inputJson = json();
                jsonput(inputJson,"parentOptionClassJson",parentOptionClassJson);
                jsonput(inputJson,"tierIndentationJson",tierIndentationJson);
                reponse = commerce.collectHybridTierDetails_c(inputJson);
                sbappend(rowStr,jsonget(reponse,"priceTableRowStr","string",""));
                jsonput(parentOptionClassJson,"tierPricesStr",jsonget(reponse,"tierPricesStr","string",""));
            }
            else{
                sbappend(rowStr,rowSecSep,skuDisplayName,"*_*",quantityStr,"*_*",unitFee,"*_*",monthly,"*_*",fee,"*_*",implementation);
            }
            //Handled in a different scenario under perOccurence table
            /*//----------------- Handling oneTimeTable --------------------------
            if(jsonget(rowPricesJson,"currentTable","string","") == "perOccurrenceTable"){
                unit = jsonget(currentPriceDataJson,"uOM","string","");
                if(jsonget(actualSKUTableJson,"FeeTableUnit","string","") <> ""){
                    unit = jsonget(actualSKUTableJson,"FeeTableUnit","string","");
                }
                rowStr = stringbuilder();
                sbappend(rowStr,rowSecSep,skuDisplayName,"*_*",quantityStr,"*_*",unit,"*_*",fee,"*_*",implementation);
            }
            //----------------- Handling oneTimeTable --------------------------*/
            
            if((jsonget(actualSKUNumberPriceJson,"tierDisplayType","string","") <> "n")){
                sbappend(priceTableRowStr,sbtostring(rowStr));
            }
        }
        //Added : Bibin Sajeevan :  25 September 2025 :  Handling Rebate SKUs ----------------------
        elif(NOT isnull(jsonget(currentPriceDataJson,"totalRebateNetAmountRollup"))){
            rowStr = stringbuilder();
            unitFee = jsonget(currentPriceDataJson,"totalRebatefeeNetPriceRollup","string","0.0");
            monthly = jsonget(currentPriceDataJson,"totalRebateNetAmountRollup","string","0.0");
            feeImplTotal = 0;
            fee = jsonget(rowPricesJson,"fee","string","n/a");
            implementation = jsonget(rowPricesJson,"implementation","string","n/a");
            //Added for Confidence model to skip in pricing as this is a rebate which will apply for next contract year
            if(jsonget(actualSKUTableJson,"DisplayType","string","") <> "SIP"){
                jsonput(rowPricesJson,"monthlyTotal",jsonget(currentPriceDataJson,"totalRebateNetAmountRollup","float",0.0));
                jsonput(rowPricesJson,"listTotal",jsonget(currentPriceDataJson,"totalRebateNetAmountRollup","float",0.0));
            }
            //Added : 13 February 2026 : Discount Assumption Prices
            jsonput(rowPricesJson,"licenseListTotal",jsonget(currentPriceDataJson,"totalLicenseListAmountRollup","float",0.0));
            jsonput(rowPricesJson,"implListTotal",jsonget(currentPriceDataJson,"totalImplementationListAmountRollup","float",0.0));
            //Added : 13 February 2026 : Discount Assumption Prices
            quantityStr = jsonget(currentPriceDataJson,"_price_quantity","string","1");
            if(quantityStr == "0"){
                quantityStr = "-";
            }
            rebateSKUJson = jsonpathgetsingle(childSKUsJson,"$.arrayPacket[?(@.feeType_l_c == 'Rebate' && @._part_number == '"+actualSKUNumber+"')]","json",json());
            //Added : 25 Feb 2026 : To handle Quote value for Rebate SKUs
            displayNetPrice = jsonget(rebateSKUJson,"displayNetPrice_l_c","string","");
            listPrice = jsonget(rebateSKUJson,"listPrice_l","string","");
            if(find(displayNetPrice,"$",1,2) == -1){
                unitFee = displayNetPrice;
                monthly = displayNetPrice;
                if(displayNetPrice == "Quote"){
                    displayListPrice = jsonget(rebateSKUJson,"displayListPrice_l_c","string","");
                    if(atof(listPrice) <> 0 AND displayListPrice <> "Quote"){
                        unitFee = listPrice;
                    }
                    jsonput(rowPricesJson,"quoteMinimum",true);
                }    
            }
            sbappend(rowStr,rowSecSep,skuDisplayName,"*_*",quantityStr,"*_*",unitFee,"*_*",monthly,"*_*",fee,"*_*",implementation);
            if((jsonget(actualSKUNumberPriceJson,"tierDisplayType","string","") <> "n")){
                sbappend(priceTableRowStr,sbtostring(rowStr));
            }
        }
        //Added : Bibin Sajeevan : 25 September 2025 : Handling Rebate SKUs --------------------
        //####################### Handling Normal Option Classes  #########################
        
        //####################### Displaying Parent Option Class of Standard SKU  #########################
        if(jsonget(lvl1PriceTablesJson,"displayStdSKU","boolean",false)){
            priceTableRowStr = stringbuilder();
            jsonremove(parentOptionClassJson,"tierPricesStr");
            if(skuDisplayName <> ""){
                //sbappend(priceTableRowStr,rowSecSep,skuDisplayName);
                sbappend(priceTableRowStr,jsonget(rowPricesJson,"displayStdSKUHeader","string",""));
            }
        }
        //####################### Displaying Parent Option Class of Standard SKU  #########################
        
        //######################## Third Party Header ######################################
        if(NOT jsonget(lvl1PriceTablesJson,"thirdPartyTable","boolean",true) AND jsonget(proposalDataJson,"Table","string","") <> ""){
            thirdPartyHeader = "@_@"+jsonget(proposalDataJson,"Table","string","")+sbtostring(priceTableRowStr);
            priceTableRowStr = stringbuilder();
            jsonput(lvl1PriceTablesJson,"thirdPartyTable",true);
            sbappend(priceTableRowStr,thirdPartyHeader);
        }
        //######################## Third Party Header ######################################
        
        //######################## Skipping Group Minimum Products from Calculations ###################################
        if(jsonget(currentPriceDataJson,"GroupMinParticipator","boolean",false)){
            jsonremove(rowPricesJson,"monthlyTotal");
            jsonremove(rowPricesJson,"listTotal");
        }
        //######################## Skipping Group Minimum Products from Calculations ###################################
            
        //####################### Decides the table to store the sku #########################
        if(NOT isnull(currentTable)){
            if(jsonget(lvl1PriceTablesJson,"hasAPriorHeader","boolean",false) AND actualSKUNumber <> ""){
                if(startswith(sbtostring(priceTableRowStr),"@_@")){
                    newPriceTableRowStr = sbtostring(priceTableRowStr);
                    newPriceTableRowStr = replace(newPriceTableRowStr,"@_@","!_!",1);
                    priceTableRowStr = stringbuilder(newPriceTableRowStr);
                    
                }
                jsonremove(lvl1PriceTablesJson,"hasAPriorHeader");
            } 
            jsonput(lvl1PriceTablesJson,currentTable,jsonget(lvl1PriceTablesJson,currentTable,"string","")+sbtostring(priceTableRowStr));
            jsonput(parentOptionClassJson,"priceTableRowStr",sbtostring(priceTableRowStr));
            jsonput(lvl1PriceTablesJson,currentTable+"MonthlyTotal",jsonget(lvl1PriceTablesJson,currentTable+"MonthlyTotal","float",0.0)+jsonget(rowPricesJson,"monthlyTotal","float",0.0));
            jsonput(lvl1PriceTablesJson,currentTable+"ListTotal",jsonget(lvl1PriceTablesJson,currentTable+"ListTotal","float",0.0)+jsonget(rowPricesJson,"listTotal","float",0.0));
            jsonput(lvl1PriceTablesJson,currentTable+"feeImplTotal",jsonget(lvl1PriceTablesJson,currentTable+"feeImplTotal","float",0.0)+jsonget(rowPricesJson,"licenseTotal","float",0.0)+jsonget(rowPricesJson,"implTotal","float",0.0));
            jsonput(lvl1PriceTablesJson,currentTable+"LicenseTotal",jsonget(lvl1PriceTablesJson,currentTable+"LicenseTotal","float",0.0)+jsonget(rowPricesJson,"licenseTotal","float",0.0));
            jsonput(lvl1PriceTablesJson,currentTable+"ImplTotal",jsonget(lvl1PriceTablesJson,currentTable+"ImplTotal","float",0.0)+jsonget(rowPricesJson,"implTotal","float",0.0));
            //Added : Bibin Sajeevan : 13 February 2026 : Collecting Prices for Discount Assumption
            jsonput(lvl1PriceTablesJson,currentTable+"ImplListTotal",jsonget(lvl1PriceTablesJson,currentTable+"ImplListTotal","float",0.0)+jsonget(rowPricesJson,"implListTotal","float",0.0));
            jsonput(lvl1PriceTablesJson,currentTable+"LicenseListTotal",jsonget(lvl1PriceTablesJson,currentTable+"LicenseListTotal","float",0.0)+jsonget(rowPricesJson,"licenseListTotal","float",0.0));
	    //Added : Bibin Sajeevan : 06 April 2026 : Discount Assumption Annual Price
            jsonput(lvl1PriceTablesJson, currentTable + "AnnualListTotal", jsonget(lvl1PriceTablesJson, currentTable + "AnnualListTotal", "float", 0.0) + jsonget(rowPricesJson, "annualListTotal", "float", 0.0));
	    //Added : Bibin Sajeevan : 06 April 2026 : Discount Assumption Annual Price
            //Added : Bibin Sajeevan : 13 February 2026 : Collecting Prices for Discount Assumption
            jsonput(lvl1PriceTablesJson,currentTable+"AnnualTotal",jsonget(lvl1PriceTablesJson,currentTable+"AnnualTotal","float",0.0)+jsonget(rowPricesJson,"annualTotal","float",0.0));
            jsonput(lvl1PriceTablesJson,currentTable+"TotalFee",jsonget(lvl1PriceTablesJson,currentTable+"TotalFee","float",0.0)+jsonget(rowPricesJson,"totalFee","float",0.0));
            jsonput(lvl1PriceTablesJson,"currentTable",currentTable);
            
        }
        else{
            jsonput(lvl1PriceTablesJson,"currentTable","licenseFeeTable");
        }
        //####################### Decides the table to store the sku ###########################
        
        //####################### Remapping QRT tier to nonBranch SKU ##########################
        if(jsonarraysize(nonBranchSKUJsonArray) > 0){
            proposalTierDisplayJsonArray = nonBranchSKUJsonArray;
        }
        //####################### Remapping QRT tier to nonBranch SKU ##########################
        
    }
    
    if(skuDisplayName <> ""){
        skuDisplayName = indentationStr+skuDisplayName;
    }
    
    //if(jsonget(lineDetailsJson,"_line_item_type") == "Standard Item" AND jsonget(lineDetailsJson,"oRCL_ABO_ActionCode_l") <> "NO_UPDATE" AND jsonget(lineDetailsJson,"oRCL_ABO_ActionCode_l") <> "DELETE"){ //Commented : 02 June 2025 : Using custom action code attribute
    if(jsonget(lineDetailsJson,"_line_item_type") == "Standard Item" AND jsonget(lineDetailsJson,"customActionCode_l_c") <> "NO_UPDATE" AND jsonget(lineDetailsJson,"customActionCode_l_c") <> "DELETE"){
    	//Added : Bibin Sajeevan : 24 February 2026 : Custom SKU Config
        if(jsonget(lineDetailsJson,"parentDocNumber_l","string","") == jsonget(modelJson,"customConfigDoc","string","")){
            custProposalDataJson = jsonget(customTableDetailsJson,partNumber,"json",json());
            inputJson = json();
            jsonput(inputJson,"custProposalDataJson",custProposalDataJson);
            jsonput(inputJson,"partNumber",partNumber);
            jsonput(inputJson,"custConfigDataJson",custConfigDataJson);
            jsonput(inputJson,"lineDetailsJson",lineDetailsJson);
            custConfigDataJson = commerce.collectCustomSKUDetails_c(inputJson);
        }
        //Added : Bibin Sajeevan : 24 February 2026 : Custom SKU Config
        //####################### Handling Inclusions  ######################
        if(jsonget(proposalDataJson,"DisplayCategory","string","") <> "" OR jsonget(lineDetailsJson,"displayNetPrice_l_c") == "Included" OR NOT isnull(jsonget(parentOptionClassJson,"aggrMonthlyProdSkuNumbers")) OR NOT isnull(jsonget(parentOptionClassJson,"aggrLicenseProdSkuNumbers")) OR NOT isnull(jsonget(parentOptionClassJson,"aggrImplementationProdSkuNumbers")) OR NOT isnull(jsonget(parentOptionClassJson,"aggrAnnualProdSkuNumbers"))){
            //################ Skipping aggregated Products from Pricing Table ###################
            if(jsonpathcheck(lvl1PriceTablesJson,"$.aggregatedProductsSKU[?(@ == '"+partNumber+"')]")){
                continue;
            }
            //################ Skipping aggregated Products from Pricing Table ###################    
            if(skuDisplayName == ""){
                continue;
            }
            /*complSKUArr = split(jsonget(proposalDataJson,"ComplSKU","string",""),",");
            complSKUJson = jsonget(modelJson,"inclusionComplSKU","json",json());
            complIndices = range(sizeofarray(complSKUArr));
            for index in complIndices{
                if(complSKUArr[index] <> ""){
                    jsonput(complSKUJson,complSKUArr[index],partNumber);
                }
            }
            if(NOT isempty(complSKUArr)){
                jsonput(modelJson,"inclusionComplSKU",complSKUJson);
            }
            if(NOT isnull(jsonget(complSKUJson,partNumber))){
                continue;
            }*/
            inputJson = json();
            jsonput(inputJson,"proposalDataJson",proposalDataJson);
            jsonput(inputJson,"modelJson",modelJson);
            jsonput(inputJson,"partNumber",partNumber);
            jsonput(inputJson,"function","inclusionComplSKU");
            response = util.util_checkComplementary(inputJson);
            modelJson = jsonget(response,"modelJson","json",modelJson);
            if(jsonget(response,"isCompl","boolean",false)){
                continue;
            } 
            inputJson = json();
            jsonput(inputJson,"proposalDataJson",proposalDataJson);
            jsonput(inputJson,"parentOptionClassJson",parentOptionClassJson);
            jsonput(inputJson,"lvl1PriceTablesJson",lvl1PriceTablesJson);
            jsonput(inputJson,"skuDisplayName",skuDisplayName);
            jsonput(inputJson,"partNumber",partNumber);
            jsonput(inputJson,"lineDetailsJson",lineDetailsJson);
            responseJson = commerce.constructInclusions_c(inputJson);
            parentOptionClassJson = jsonget(responseJson,"parentOptionClassJson","json",parentOptionClassJson);
            lvl1PriceTablesJson = jsonget(responseJson,"lvl1PriceTablesJson","json",lvl1PriceTablesJson);
            
        }
        if(NOT isnull(jsonget(inclusionsJson,partNumber))){
            inputJson = json();
            jsonput(inputJson,"proposalDataJson",proposalDataJson);
            jsonput(inputJson,"parentOptionClassJson",parentOptionClassJson);
            jsonput(inputJson,"lvl1PriceTablesJson",lvl1PriceTablesJson);
            jsonput(inputJson,"skuDisplayName",skuDisplayName);
            jsonput(inputJson,"partNumber",partNumber);
            jsonput(inputJson,"lineDetailsJson",lineDetailsJson);
            jsonput(inputJson,"inclusionsJson",inclusionsJson);
            responseJson = commerce.constructInclusions_c(inputJson);
            parentOptionClassJson = jsonget(responseJson,"parentOptionClassJson","json",parentOptionClassJson);
            lvl1PriceTablesJson = jsonget(responseJson,"lvl1PriceTablesJson","json",lvl1PriceTablesJson);
        }
        //####################### Handling Inclusions  ######################
        
        //####################### Substracting Group Minimum Aggregating SKUs ########################
        if(jsonget(lineDetailsJson,"isGroupMinimumContributor_l_c","boolean",false) AND jsonpathcheck(parentOptionClassJson,"$.aggrMonthlyProdSkuNumbers[?(@ == '"+partNumber+"')]")){ 
            netAmount = jsonget(lineDetailsJson,"netAmount_l","float",0.0);
            listAmount = jsonget(lineDetailsJson,"listAmount_l","float",0.0);
            currentTable = jsonget(proposalDataJson,"PriceTable");
            jsonput(lvl1PriceTablesJson,currentTable+"MonthlyTotal",jsonget(lvl1PriceTablesJson,currentTable+"MonthlyTotal","float",0.0)-netAmount);
            jsonput(lvl1PriceTablesJson,currentTable+"ListTotal",jsonget(lvl1PriceTablesJson,currentTable+"ListTotal","float",0.0)-listAmount);
        }
        //####################### Substracting Group Minimum Aggregating SKUs ########################
        
        //####################### 24 October 2025 :Handling Formula Pricing Aggregates #################################
        //if(jsonget(lineDetailsJson,"formulaInputSKUs_l_c","string","") <> ""){
        //    aggregatedProductsSKUStr = jsonget(lineDetailsJson,"formulaInputSKUs_l_c","string","");
        //    aggregatedProductsSKUStr = "[\""+ replace(aggregatedProductsSKUStr,",","\",\"") + "\"]";
            //jsonput(parentOptionClassJson,"aggrProdSkuNumber",jsonarray(aggregatedProductsSKUStr));
        //    jsonput(parentOptionClassJson,"aggrMonthlyProdSkuNumbers",jsonarray(aggregatedProductsSKUStr));
        //}
        //####################### 24 October 2025 :Handling Formula Pricing Aggregates #################################
        
        //####################### Handling Standard SKU Aggregations ##################
        if(NOT isnull(jsonget(currentPriceDataJson,"aggregateNetAmount"))){
            priceTableRowStr = stringbuilder();
            monthly = jsonget(currentPriceDataJson,"aggregateNetAmount","string","0.0");
            unitFee = monthly;
            if(jsonget(currentPriceDataJson,"TypeOfAggregation") == "Aggregate List Price"){
                unitFee = jsonget(currentPriceDataJson,"aggregateNetPrice","string","0.0");
            }
            //Jira 1702
            displayNetPrice = jsonget(lineDetailsJson,"displayNetPrice_l_c","string","");
            if(find(displayNetPrice,"$",0,1) == -1){
                unitFee = displayNetPrice;
                monthly = displayNetPrice;    
            }
            //Jira 1702
            
            aboJson = jsonget(lineDetailsJson,"aBOJson_l_c","json",json());
            aggregateJsonArray = jsonpathgetsingle(aboJson,"$.currentData.proposalTierDisplay_l_c","jsonarray",jsonarray());
            isQrtSKU = false;
            if(findinarray(qrtFlags,jsonget(proposalDataJson,"DisplayFormat","string","")) <> -1){
                isQrtSKU = true;
            }
            if(jsonget(parentOptionClassJson,"noOfLicenseSKUs","integer",0) == 1 OR jsonget(parentOptionClassJson,"noOfImplementationSKUs","integer",0) == 1){
                inputJson = json();
                jsonput(inputJson,"parentOptionClassJson",parentOptionClassJson);
                jsonput(inputJson,"lvl1PriceTablesJson",lvl1PriceTablesJson);
                jsonput(inputJson,"lineDetailsJson",lineDetailsJson);
                jsonput(inputJson,"proposalDataJson",proposalDataJson);
                jsonput(inputJson,"unitFee",unitFee);
                jsonput(inputJson,"monthly",monthly);
                response = commerce.mergeStandardAggregateDetails_c(inputJson);
                parentOptionClassJson = jsonget(response,"parentOptionClassJson","json",json());
                lvl1PriceTablesJson = jsonget(response,"lvl1PriceTablesJson","json",json());
            }
            //Added : Bibin Sajeevan : 25 December 2025 : Aggregate Tiers : IBS (9952-10008-9621)
            // elif(jsonarraysize(aggregateJsonArray) > 1) Updated : Bibin Sajeevan : QRT Aggregate SKU (976-4420-56708)
            elif(jsonarraysize(aggregateJsonArray) > 1 AND NOT isQrtSKU){
                inputJson = json();
                jsonput(inputJson,"aggregateJsonArray",aggregateJsonArray);
                jsonput(inputJson,"lineDetailsJson",lineDetailsJson);
                jsonput(inputJson,"currentPriceDataJson",currentPriceDataJson);
                jsonput(inputJson,"proposalDataJson",proposalDataJson);
                jsonput(inputJson,"tierIndentationJson",tierIndentationJson);
                jsonput(inputJson,"lvl1PriceTablesJson",lvl1PriceTablesJson);
                
                reponse = commerce.collectAggregateDetails_c(inputJson);
                
                sbappend(priceTableRowStr,rowSecSep,skuDisplayName,jsonget(reponse,"priceTableRowStr","string",""));
                jsonput(parentOptionClassJson,"tierPricesStr",jsonget(reponse,"tierPricesStr","string",""));
                lvl1PriceTablesJson = jsonget(reponse,"lvl1PriceTablesJson","json",json());
            }
            //Added : Bibin Sajeevan : 25 December 2025 : Aggregate Tiers
            else{
                sbappend(priceTableRowStr,rowSecSep,skuDisplayName,"*_*",jsonget(lineDetailsJson,"_price_quantity","string","0"),"*_*",unitFee,"*_*",monthly,"*_*","n/a","*_*","n/a");
            }
            jsonput(parentOptionClassJson,"standardAggregateSKU",sbtostring(priceTableRowStr));
            jsonput(parentOptionClassJson,"aggrUnitFee",unitFee);
            jsonput(parentOptionClassJson,"aggrMonthlyFee",monthly);
            jsonput(parentOptionClassJson,"aggrMonthlySKUDisplay",skuDisplayName);
            aggrSKUDisplayNames = jsonget(parentOptionClassJson,"aggrSKUDisplayNames","jsonarray",jsonarray());
            jsonarrayappend(aggrSKUDisplayNames,skuDisplayName);
            jsonput(parentOptionClassJson,"aggrSKUDisplayNames",aggrSKUDisplayNames);
            if(NOT isnull(jsonget(currentPriceDataJson,"aggrProdSkuNumber"))){
                aggregatedProductsSKUStr = replace(jsonget(currentPriceDataJson,"aggrProdSkuNumber","string",""),",","",1);
                aggregatedProductsSKUStr = "[\""+ replace(aggregatedProductsSKUStr,",","\",\"") + "\"]";
                //jsonput(parentOptionClassJson,"aggrProdSkuNumber",jsonarray(aggregatedProductsSKUStr));
                jsonput(parentOptionClassJson,"aggrMonthlyProdSkuNumbers",jsonarray(aggregatedProductsSKUStr));
            }
            
            currentTable = jsonget(proposalDataJson,"PriceTable");
            jsonput(lvl1PriceTablesJson,"currentTable",currentTable);
            if(NOT jsonget(lineDetailsJson,"isGroupMinimumContributor_l_c","boolean",false)){ // Skipping aggregate price if SKU is participating in Group Minimum
                jsonput(lvl1PriceTablesJson,currentTable+"MonthlyTotal",jsonget(lvl1PriceTablesJson,currentTable+"MonthlyTotal","float",0.0)+jsonget(currentPriceDataJson,"aggregateNetAmount","float",0.0));
                jsonput(lvl1PriceTablesJson,currentTable+"ListTotal",jsonget(lvl1PriceTablesJson,currentTable+"ListTotal","float",0.0)+jsonget(currentPriceDataJson,"aggregateListAmount","float",0.0));
            }
        }
        elif(NOT isnull(jsonget(currentPriceDataJson,"aggregateLicenseFee"))){
            priceTableRowStr = stringbuilder();
            license = jsonget(currentPriceDataJson,"aggregateLicenseFee","string","0.0");
            //Jira 1702
            displayNetPrice = jsonget(lineDetailsJson,"displayNetPrice_l_c","string","");
            if(find(displayNetPrice,"$",0,1) == -1){
                license = displayNetPrice;    
            }
            //Jira 1702
            //sbappend(priceTableRowStr,rowSecSep,skuDisplayName,"*_*",jsonget(lineDetailsJson,"_price_quantity","string","0"),"*_*","n/a","*_*","n/a","*_*",license,"*_*","n/a");
            if(jsonget(parentOptionClassJson,"noOfImplementationSKUs","integer",0) == 1 OR jsonget(parentOptionClassJson,"noOfMonthlySKUs","integer",0) == 1){
                inputJson = json();
                jsonput(inputJson,"parentOptionClassJson",parentOptionClassJson);
                jsonput(inputJson,"lvl1PriceTablesJson",lvl1PriceTablesJson);
                jsonput(inputJson,"lineDetailsJson",lineDetailsJson);
                jsonput(inputJson,"proposalDataJson",proposalDataJson);
                jsonput(inputJson,"license",license);
                response = commerce.mergeStandardAggregateDetails_c(inputJson);
                parentOptionClassJson = jsonget(response,"parentOptionClassJson","json",json());
                lvl1PriceTablesJson = jsonget(response,"lvl1PriceTablesJson","json",json());    
            }
            else{
                sbappend(priceTableRowStr,rowSecSep,skuDisplayName,"*_*",jsonget(lineDetailsJson,"_price_quantity","string","0"),"*_*","n/a","*_*","n/a","*_*",license,"*_*","n/a");
            }
            jsonput(parentOptionClassJson,"standardAggregateSKU",sbtostring(priceTableRowStr));
            jsonput(parentOptionClassJson,"aggrLicense",license);
            if(NOT isnull(jsonget(currentPriceDataJson,"aggrProdSkuNumber"))){
                aggregatedProductsSKUStr = replace(jsonget(currentPriceDataJson,"aggrProdSkuNumber","string",""),",","",1);
                aggregatedProductsSKUStr = "[\""+ replace(aggregatedProductsSKUStr,",","\",\"") + "\"]";
                //jsonput(parentOptionClassJson,"aggrProdSkuNumber",jsonarray(aggregatedProductsSKUStr));
                jsonput(parentOptionClassJson,"aggrLicenseProdSkuNumbers",jsonarray(aggregatedProductsSKUStr));
            }
            currentTable = jsonget(proposalDataJson,"PriceTable");
            jsonput(lvl1PriceTablesJson,"currentTable",currentTable);
            jsonput(lvl1PriceTablesJson,currentTable+"LicenseTotal",jsonget(lvl1PriceTablesJson,currentTable+"LicenseTotal","float",0.0)+jsonget(currentPriceDataJson,"aggregateLicenseFee","float",0.0));
            jsonput(lvl1PriceTablesJson,currentTable+"feeImplTotal",jsonget(lvl1PriceTablesJson,currentTable+"feeImplTotal","float",0.0)+jsonget(currentPriceDataJson,"aggregateLicenseFee","float",0.0));
            //Added : Bibin Sajeevan : 13 February 2026 : Discount Assumption Prices
            jsonput(lvl1PriceTablesJson,currentTable+"LicenseListTotal",jsonget(lvl1PriceTablesJson,currentTable+"LicenseListTotal","float",0.0)+jsonget(currentPriceDataJson,"aggregateLicenseListAmount","float",0.0));
            //Added : Bibin Sajeevan : 13 February 2026 : Discount Assumption Prices
        }
        //--------- Bibin Sajeevan : 25 September 2025 : Annual Aggregations ----------------------
        elif(NOT isnull(jsonget(currentPriceDataJson,"aggregateAnnualNetAmount"))){
            priceTableRowStr = stringbuilder();
            annual = jsonget(currentPriceDataJson,"aggregateAnnualNetAmount","string","0.0");
            displayNetPrice = jsonget(lineDetailsJson,"displayNetPrice_l_c","string","");
            if(find(displayNetPrice,"$",0,1) == -1){
                annual = displayNetPrice;    
            }
            if(jsonget(parentOptionClassJson,"noOfImplementationSKUs","integer",0) == 1){
                inputJson = json();
                jsonput(inputJson,"parentOptionClassJson",parentOptionClassJson);
                jsonput(inputJson,"lvl1PriceTablesJson",lvl1PriceTablesJson);
                jsonput(inputJson,"lineDetailsJson",lineDetailsJson);
                jsonput(inputJson,"proposalDataJson",proposalDataJson);
                jsonput(inputJson,"annual",annual);
                response = commerce.mergeStandardAggregateDetails_c(inputJson);
                parentOptionClassJson = jsonget(response,"parentOptionClassJson","json",json());
                lvl1PriceTablesJson = jsonget(response,"lvl1PriceTablesJson","json",json());    
            }
            else{
                sbappend(priceTableRowStr,rowSecSep,skuDisplayName,"*_*",jsonget(lineDetailsJson,"_price_quantity","string","0"),"*_*","n/a","*_*","n/a","*_*",annual,"*_*","n/a");
            }
            jsonput(parentOptionClassJson,"standardAggregateSKU",sbtostring(priceTableRowStr));
            jsonput(parentOptionClassJson,"aggrAnnual",annual);
            aggrSKUDisplayNames = jsonget(parentOptionClassJson,"aggrSKUDisplayNames","jsonarray",jsonarray());
            jsonarrayappend(aggrSKUDisplayNames,skuDisplayName);
            jsonput(parentOptionClassJson,"aggrSKUDisplayNames",aggrSKUDisplayNames);
            if(NOT isnull(jsonget(currentPriceDataJson,"aggrProdSkuNumber"))){
                aggregatedProductsSKUStr = replace(jsonget(currentPriceDataJson,"aggrProdSkuNumber","string",""),",","",1);
                aggregatedProductsSKUStr = "[\""+ replace(aggregatedProductsSKUStr,",","\",\"") + "\"]";
                //jsonput(parentOptionClassJson,"aggrProdSkuNumber",jsonarray(aggregatedProductsSKUStr));
                jsonput(parentOptionClassJson,"aggrAnnualProdSkuNumbers",jsonarray(aggregatedProductsSKUStr));
            }
            currentTable = jsonget(proposalDataJson,"PriceTable");
            jsonput(lvl1PriceTablesJson,"currentTable",currentTable);
            //Updated : Bibin Sajeevan : 13 February 2026 : Annual Fee should go into new Annual Total
            //jsonput(lvl1PriceTablesJson,currentTable+"LicenseTotal",jsonget(lvl1PriceTablesJson,currentTable+"LicenseTotal","float",0.0)+jsonget(currentPriceDataJson,"aggregateAnnualNetAmount","float",0.0));
            //jsonput(lvl1PriceTablesJson,currentTable+"feeImplTotal",jsonget(lvl1PriceTablesJson,currentTable+"feeImplTotal","float",0.0)+jsonget(currentPriceDataJson,"aggregateAnnualNetAmount","float",0.0));
            jsonput(lvl1PriceTablesJson,currentTable+"AnnualTotal",jsonget(lvl1PriceTablesJson,currentTable+"AnnualTotal","float",0.0)+jsonget(currentPriceDataJson,"aggregateAnnualNetAmount","float",0.0));
			//Added : Bibin Sajeevan : 06 April 2026 : Discount Assumption Annual Price
            jsonput(lvl1PriceTablesJson, currentTable + "AnnualListTotal", jsonget(lvl1PriceTablesJson, currentTable + "AnnualListTotal", "float", 0.0) + jsonget(currentPriceDataJson, "aggregateAnnualListAmount", "float", 0.0));
			//Added : Bibin Sajeevan : 06 April 2026 : Discount Assumption Annual Price
            //Updated : Bibin Sajeevan : 13 February 2026 : Annual Fee should go into new Annual Total
        }
        //--------- Bibin Sajeevan : 25 September 2025 : Annual Aggregations ----------------------
        elif(NOT isnull(jsonget(currentPriceDataJson,"aggregateImplementationFee"))){
            priceTableRowStr = stringbuilder();
            unitFee = jsonget(parentOptionClassJson,"aggrUnitFee","string","n/a");
            monthly = jsonget(parentOptionClassJson,"aggrMonthlyFee","string","n/a");
            license = jsonget(parentOptionClassJson,"aggrLicense","string","n/a");
            implementation = jsonget(currentPriceDataJson,"aggregateImplementationFee","string","0.0");
            if(jsonget(currentPriceDataJson,"TypeOfAggregation") == "Aggregate List Price"){
                license = jsonget(currentPriceDataJson,"aggregateImplementationNetPrice","string","0.0");
            }
            //Jira 1702
            displayNetPrice = jsonget(lineDetailsJson,"displayNetPrice_l_c","string","");
            if(find(displayNetPrice,"$",0,1) == -1){
                implementation = displayNetPrice;    
            }
            //Jira 1702
            if(jsonget(proposalDataJson,"PriceTable") == "unitImplTable"){
                license = implementation;
            }
            if(jsonget(parentOptionClassJson,"noOfMonthlySKUs","integer",0) == 1 OR jsonget(parentOptionClassJson,"noOfLicenseSKUs","integer",0) == 1){
                inputJson = json();
                jsonput(inputJson,"parentOptionClassJson",parentOptionClassJson);
                jsonput(inputJson,"lvl1PriceTablesJson",lvl1PriceTablesJson);
                jsonput(inputJson,"lineDetailsJson",lineDetailsJson);
                jsonput(inputJson,"proposalDataJson",proposalDataJson);
                jsonput(inputJson,"implementation",implementation);
                response = commerce.mergeStandardAggregateDetails_c(inputJson);
                parentOptionClassJson = jsonget(response,"parentOptionClassJson","json",json());
                lvl1PriceTablesJson = jsonget(response,"lvl1PriceTablesJson","json",json());
            }
            else{
                sbappend(priceTableRowStr,rowSecSep,skuDisplayName,"*_*",jsonget(lineDetailsJson,"_price_quantity","string","0"),"*_*",unitFee,"*_*",monthly,"*_*",license,"*_*",implementation);
            }
            jsonput(parentOptionClassJson,"standardAggregateSKU",sbtostring(priceTableRowStr));
            aggrSKUDisplayNames = jsonget(parentOptionClassJson,"aggrSKUDisplayNames","jsonarray",jsonarray());
            jsonarrayappend(aggrSKUDisplayNames,skuDisplayName);
            jsonput(parentOptionClassJson,"aggrSKUDisplayNames",aggrSKUDisplayNames);
            if(NOT isnull(jsonget(currentPriceDataJson,"aggrProdSkuNumber"))){
                aggregatedProductsSKUStr = replace(jsonget(currentPriceDataJson,"aggrProdSkuNumber","string",""),",","",1);
                aggregatedProductsSKUStr = "[\""+ replace(aggregatedProductsSKUStr,",","\",\"") + "\"]";
                //if(NOT isnull(jsonget(parentOptionClassJson,"aggrProdSkuNumber"))){
                //    aggregatedProductsSKUStr = substring(jsonarraytostr(jsonget(parentOptionClassJson,"aggrProdSkuNumber","jsonarray")),0,-1) + replace(aggregatedProductsSKUStr,"[",",",1);
                //}
                //jsonput(parentOptionClassJson,"aggrProdSkuNumber",jsonarray(aggregatedProductsSKUStr));
                jsonput(parentOptionClassJson,"aggrImplementationProdSkuNumbers",jsonarray(aggregatedProductsSKUStr));
            }
            currentTable = jsonget(proposalDataJson,"PriceTable");
            jsonput(lvl1PriceTablesJson,"currentTable",currentTable);
            jsonput(lvl1PriceTablesJson,currentTable+"feeImplTotal",jsonget(lvl1PriceTablesJson,currentTable+"feeImplTotal","float",0.0)+jsonget(currentPriceDataJson,"aggregateImplementationFee","float",0.0));
            jsonput(lvl1PriceTablesJson,currentTable+"ImplTotal",jsonget(lvl1PriceTablesJson,currentTable+"ImplTotal","float",0.0)+jsonget(currentPriceDataJson,"aggregateImplementationFee","float",0.0));
            //Added : Bibin Sajeevan : 13 February 2026 : Discount Assumption Prices
            jsonput(lvl1PriceTablesJson,currentTable+"ImplListTotal",jsonget(lvl1PriceTablesJson,currentTable+"ImplListTotal","float",0.0)+jsonget(currentPriceDataJson,"aggregateImplementationListAmount","float",0.0));
            //Added : Bibin Sajeevan : 13 February 2026 : Discount Assumption Prices
        }
        //####################### Handling Standard SKU Aggregations ##################
        
        //####################### Display Standard SKU  ###########################
        elif(jsonget(lvl1PriceTablesJson,"displayStdSKU","boolean",false) AND NOT jsonpathcheck(lineDetailsJson,"$..[?(@.grpMinSKUDetails_l_c != '')]")){
            //################ Skipping aggregated Products from Pricing Table ###################
            if(jsonpathcheck(parentOptionClassJson,"$.aggrMonthlyProdSkuNumbers[?(@ == '"+partNumber+"')]") OR jsonpathcheck(parentOptionClassJson,"$.aggrLicenseProdSkuNumbers[?(@ == '"+partNumber+"')]") OR jsonpathcheck(parentOptionClassJson,"$.aggrImplementationProdSkuNumbers[?(@ == '"+partNumber+"')]") OR jsonpathcheck(parentOptionClassJson,"$.aggrAnnualProdSkuNumbers[?(@ == '"+partNumber+"')]")){
                continue;
            }
            if(jsonpathcheck(lvl1PriceTablesJson,"$.aggregatedProductsSKU[?(@ == '"+partNumber+"')]")){
                continue;
            }
            //################ Skipping aggregated Products from Pricing Table ###################    
            if(skuDisplayName == ""){
                continue;    
            }
            //################ Skipping TCV SKUs : Endpoint Exchange ###################################
            if(jsonget(lineDetailsJson,"feeType_l_c","string","") == "TCV"){
                continue;
            }
            //################ Skipping TCV SKUs : Endpoint Exchange ###################################
    
            inputJson = json();
            jsonput(inputJson,"lineDetailsJson",lineDetailsJson);
            jsonput(inputJson,"proposalDataJson",proposalDataJson);
            jsonput(inputJson,"currentPriceDataJson",currentPriceDataJson);
            jsonput(inputJson,"rowSecSep",rowSecSep);
            jsonput(inputJson,"skuDisplayName",skuDisplayName);
            jsonput(inputJson,"lvl1PriceTablesJson",lvl1PriceTablesJson);
            jsonput(inputJson,"tierIndentationJson",tierIndentationJson);
            jsonput(inputJson,"parentOptionClassJson",parentOptionClassJson);
            response = commerce.displayStandardSKU_c(inputJson);
            parentOptionClassJson = jsonget(response,"parentOptionClassJson","json",parentOptionClassJson);
            lvl1PriceTablesJson = jsonget(response,"lvl1PriceTablesJson","json",lvl1PriceTablesJson);
            jsonput(parentOptionClassJson,"displayStdSKUStr",jsonget(parentOptionClassJson,"displayStdSKUStr","string","")+jsonget(response,"displayStdSKUStr","string",""));
        }
        //#######################  Display Standard SKU  ############################
        
        //################ Model Related Fee Schedule Table Generation #######################
        if(jsonget(proposalDataJson,"DisplayFormat","string","") == "Miscellaneous Fees"){
            
            inputJson = json();
            jsonput(inputJson,"proposalDataJson",proposalDataJson);
            jsonput(inputJson,"modelJson",modelJson);
            jsonput(inputJson,"partNumber",partNumber);
            jsonput(inputJson,"function","complSKU");
            response = util.util_checkComplementary(inputJson);
            modelJson = jsonget(response,"modelJson","json",modelJson);
            if(jsonget(response,"isCompl","boolean",false)){
                continue;
            }
            inputJson = json();
            jsonput(inputJson,"lvl1PriceTablesJson",lvl1PriceTablesJson);
            jsonput(inputJson,"proposalDataJson",proposalDataJson);
            jsonput(inputJson,"modelJson",modelJson);
            jsonput(inputJson,"lineDetailsJson",lineDetailsJson);
            jsonput(inputJson,"indentationStr",indentationStr);
            jsonput(inputJson,"skuDisplayName",skuDisplayName);
            response = commerce.constructMiscellaneousFees_c(inputJson);
            lvl1PriceTablesJson= jsonget(inputJson,"lvl1PriceTablesJson","json",lvl1PriceTablesJson);
            modelJson = jsonget(inputJson,"modelJson","json",modelJson);
        }
        //################ Model Related Fee Schedule Table Generation #######################
        
        //################ Adding QRT SKUs to Fee Table ###################
        
        if(findinarray(qrtFlags,jsonget(proposalDataJson,"DisplayFormat","string","")) <> -1){
           
            inputJson = json();
            jsonput(inputJson,"proposalDataJson",proposalDataJson);
            jsonput(inputJson,"modelJson",modelJson);
            jsonput(inputJson,"partNumber",partNumber);
            jsonput(inputJson,"function","qRTComplSKU");
            response = util.util_checkComplementary(inputJson);
            modelJson = jsonget(response,"modelJson","json",modelJson);
            if(jsonget(response,"isCompl","boolean",false)){
                continue;
            }

            rowSecSep = "@_@";
            altSkuDisplayName = jsonget(proposalDataJson,"AltSKUDisplay","string","");
            altSkuDisplayName = jsonget(qrtDisplayNameJson,partNumber,"string",altSkuDisplayName);
            altSkuDisplayName = replace(replace(replace(altSkuDisplayName,"&","&amp;"),"<","&lt;"),">","&gt;");
            if(altSkuDisplayName == ""){
                altSkuDisplayName = nonIndentSkuDisplayName;
                
            }
            aboJson = jsonget(lineDetailsJson,"aBOJson_l_c","json",json());
            proposalTierDisplayJsonArray = jsonpathgetsingle(aboJson,"$.currentData.proposalTierDisplay_l_c","jsonarray",jsonarray());
            billingFrequency = jsonget(lineDetailsJson,"billingFrequency_l_c","string","");
            qrtType = jsonget(proposalDataJson,"DisplayFormat","string","");
            discount = jsonget(lineDetailsJson,"customDiscountValue_l","string","0.0");
            if(jsonget(currentPriceDataJson,"minApplying","boolean",false)){
                discount = jsonget(lineDetailsJson,"minimumDiscount_l_c","string","0.0");
                if(discount == ""){
                    discount = "0.0";
                }
            }
            if((qrtType == "QRT3" AND discount == "0.0") OR (qrtType == "QRT1" AND billingFrequency <> "Monthly")){
                qrtType = "";
            }
            if(jsonget(currentPriceDataJson,"TierDiscApplicable","string","Y") == "N" AND jsonget(lineDetailsJson,"customDiscountValue_l","string","0.0") <> "0.0" AND NOT jsonget(modelJson,"tierDiscountNotApplicable","boolean",false)){
                jsonput(modelJson,"tierDiscountNotApplicable","true");
            }
            if((jsonget(currentPriceDataJson,"TierDiscApplicable","string","Y") == "Y" OR jsonget(currentPriceDataJson,"TierDiscApplicable","string","Y") == "") AND NOT jsonget(modelJson,"tierDiscountApplicable","boolean",false)){
                jsonput(modelJson,"tierDiscountApplicable","true");
            }
            licenseProposalTierDisplayJsonArray = jsonarray();
            if(qrtType == "QRT1" AND billingFrequency == "Monthly"){
                licenseAboJson  = jsonpathgetsingle(childSKUsJson,"$.arrayPacket.[?(@.feeType_l_c == 'License')].aBOJson_l_c","json",json());
                licenseProposalTierDisplayJsonArray = jsonpathgetsingle(licenseAboJson,"$.currentData.proposalTierDisplay_l_c","jsonarray",jsonarray());
            }
            if((jsonget(currentPriceDataJson,"tierDisplayType","string","") == "n")){
                if(jsonarraysize(proposalTierDisplayJsonArray) > 0){
                    jsonarrayremove(proposalTierDisplayJsonArray,0);
                }
                if(jsonarraysize(licenseProposalTierDisplayJsonArray) > 0){
                    jsonarrayremove(licenseProposalTierDisplayJsonArray,0);
                }
                if(jsonarraysize(proposalTierDisplayJsonArray) == 0){
                    qrtType = "";
                }
            }
            qrtHeaderStr = stringbuilder();
            if(jsonget(modelJson,"QRT","string","") == "" AND qrtType <> ""){
                //jsonput(modelJson,"QRT",jsonget(modelJson,"QRT","string","")+"@_@Qualified Rate Table");
                sbappend(qrtHeaderStr,"@_@Qualified Rate Table");
                rowSecSep = "!_!";
            }
            if(NOT isnull(jsonget(lvl1PriceTablesJson,"qrtOptionClassHeader"))){
                qrtOptionClassHeader = jsonget(lvl1PriceTablesJson,"qrtOptionClassHeader","string","");
                if(rowSecSep == "!_!"){
                    qrtOptionClassHeader = replace(qrtOptionClassHeader,"@_@","!_!",1);
                }
                sbappend(qrtHeaderStr,qrtOptionClassHeader);
                //jsonput(modelJson,"QRT",jsonget(modelJson,"QRT","string","")+qrtHeaderStr);
                //jsonremove(lvl1PriceTablesJson,"qrtOptionClassHeader");
                rowSecSep = "!_!";
            }
            if(altSkuDisplayName <> ""){
                if(feeTableIndentationStr <> ""){
                    altSkuDisplayName = feeTableIndentationStr+altSkuDisplayName;
                }
            }
            if(qrtType <> ""){
                inputJson = json();
                jsonput(inputJson,"qRTType",qrtType);
                jsonput(inputJson,"rowSecSep",rowSecSep);
                jsonput(inputJson,"qRTSkuDisplayName",altSkuDisplayName);
                jsonput(inputJson,"lineDetailsJson",lineDetailsJson);
                jsonput(inputJson,"proposalDataJson",proposalDataJson);
                jsonput(inputJson,"tierIndentationJson",tierIndentationJson);
                jsonput(inputJson,"feeTableIndentationStr",feeTableIndentationStr);
                jsonput(inputJson,"proposalTierDisplayJsonArray",proposalTierDisplayJsonArray);
                jsonput(inputJson,"licenseProposalTierDisplayJsonArray",licenseProposalTierDisplayJsonArray);
                qrtStr = commerce.generateQRTString_c(inputJson);
                if(qrtStr <> ""){
                    jsonput(modelJson,"QRT",jsonget(modelJson,"QRT","string","")+sbtostring(qrtHeaderStr));
                    jsonremove(lvl1PriceTablesJson,"qrtOptionClassHeader");
                }
                jsonput(modelJson,"QRT",jsonget(modelJson,"QRT","string","")+qrtStr);
            }
        }
        //################ Adding QRT SKUs to Fee Table ###################
        
        //################ Professional Service Model #########################################
        if(jsonget(modelJson,"modelVarName","string","") == "professionalServices_m"){
            inputJson = json();
            jsonput(inputJson,"lineDetailsJson",lineDetailsJson);
            jsonput(inputJson,"fixedFeeMergeJson",fixedFeeMergeJson);
            jsonput(inputJson,"lvl1PriceTablesJson",lvl1PriceTablesJson);
            
            response = commerce.constructProfessionalServices_c(inputJson);
            lvl1PriceTablesJson = jsonget(response,"lvl1PriceTablesJson","json",lvl1PriceTablesJson);
            fixedFeeMergeJson = jsonget(response,"fixedFeeMergeJson","json",fixedFeeMergeJson);
        }
        
        //################ Professional Service Model #########################################                
    }
    //####################### Handling Subtotals ######################
    if(jsonpathcheck(lineDetailsJson,"$..[?(@.grpMinSKUDetails_l_c != '')]")){
        inputJson = json();
        jsonput(inputJson,"lineDetailsJson",lineDetailsJson);
        jsonput(inputJson,"lvl1PriceTablesJson",lvl1PriceTablesJson);
        jsonput(inputJson,"proposalDataJson",proposalDataJson);
        jsonput(inputJson,"skuDisplayName",skuDisplayName);
        jsonput(inputJson,"parentOptionClassJson",parentOptionClassJson);
        jsonput(inputJson,"rowSecSep",rowSecSep);
        response = commerce.constructGroupMinimum_c(inputJson);
        lvl1PriceTablesJson = jsonget(response,"lvl1PriceTablesJson","json",json());
        parentOptionClassJson = jsonget(response,"parentOptionClassJson","json",json());
    }
    //####################### Handling Subtotals  ######################    
}

//############# Adding last model ####################
if(modelDocNum <> ""){
    inputJson = json(); 
    jsonput(inputJson,"parentOptionClassJson",parentOptionClassJson);
    jsonput(inputJson,"lvl1PriceTablesJson",lvl1PriceTablesJson);
    reponse = commerce.updatePreviousOptionClass_c(inputJson);
    lvl1PriceTablesJson = jsonget(reponse,"lvl1PriceTablesJson","json",lvl1PriceTablesJson);
    parentOptionClassJson = jsonget(reponse,"parentOptionClassJson","json",parentOptionClassJson);
    
    if(NOT isnull(jsonget(lvl1PriceTablesJson,"printWithChildJsonArr"))){ 
        reqJson = json();
        jsonput(reqJson,"lvl1PriceTablesJson",lvl1PriceTablesJson);
        //jsonput(reqJson,"lineDetailsJson",lineDetailsJson);
        lvl1PriceTablesJson = jsonget(commerce.setPrintWithGrandChild_c(reqJson),"response","json",json());
        
    }
    
    //Updated : Bibin Sajeevan : 01 April 2026 : Fixed Fee Merge Sub Lib : QTCCPQ - 4455
    if(sizeofarray(jsonkeys(fixedFeeMergeJson)) > 0){
    	inputJson = json();
	jsonput(inputJson,"fixedFeeMergeJson",fixedFeeMergeJson);
	jsonput(inputJson,"lvl1PriceTablesJson",lvl1PriceTablesJson);
	response = commerce.mergePSFixedFees_c(inputJson);
	lvl1PriceTablesJson = jsonget(response,"lvl1PriceTablesJson","json",lvl1PriceTablesJson);
	fixedFeeMergeJson = json();
    }
    //Updated : Bibin Sajeevan : 01 April 2026 : Fixed Fee Merge Sub Lib : QTCCPQ - 4455
	
    //Added : Bibin Sajeevan : 24 February 2026 : Custom SKU Config
    if(NOT isnull(jsonget(custConfigDataJson,jsonget(lvl1PriceTablesJson,"tableHeader","string","")))){
        customSKUsDataJson = jsonget(custConfigDataJson,jsonget(lvl1PriceTablesJson,"tableHeader","string",""),"json",json());
        inputJson = json();
        jsonput(inputJson,"modelJson",modelJson);
        jsonput(inputJson,"lvl1PriceTablesJson",lvl1PriceTablesJson);
        jsonput(inputJson,"customSKUsDataJson",customSKUsDataJson);
        jsonput(inputJson,"tierIndentationJson",tierIndentationJson);
        reponse = commerce.mergeCustomSKUDetails_c(inputJson);
        jsonremove(custConfigDataJson,jsonget(lvl1PriceTablesJson,"tableHeader","string",""));
        lvl1PriceTablesJson = jsonget(reponse,"lvl1PriceTablesJson","json",lvl1PriceTablesJson);
        modelJson = jsonget(reponse,"modelJson","json",modelJson);
    }
    //Added : Bibin Sajeevan : 24 February 2026 : Custom SKU Config
    
        
    inputJson = json();
    jsonput(inputJson,"lvl1PriceTablesJson",lvl1PriceTablesJson);
    jsonput(inputJson,"transactionPriceTableJson",transactionPriceTableJson); 
    jsonput(inputJson,"modelJson",modelJson);
    if(NOT isnull(jsonget(abstractModelsJson,jsonget(modelJson,"modelVarName","string","")))){
        jsonput(inputJson,"abstractModel",jsonget(abstractModelsJson,jsonget(modelJson,"modelVarName","string","")));
        jsonput(inputJson,"pseudoModelJson",pseudoModelJson);
    } 
    reponse = commerce.collectLvl1OptionClassDetails_c(inputJson);
    lvl1PriceTablesJson = json();
    modelJson = jsonget(reponse,"modelJson","json",modelJson);
    pseudoModelJson = jsonget(reponse,"pseudoModelJson","json",pseudoModelJson);
    transactionPriceTableJson = jsonget(reponse,"transactionPriceTableJson","json",transactionPriceTableJson);
    
    //###### Adding new Tables if Custom SKU Config table is not present in the Transaction ########
    newCustTableKeys = jsonkeys(custConfigDataJson);
    if(sizeofarray(newCustTableKeys) > 0){
        for newTableHeader in newCustTableKeys{
            customSKUsDataJson = jsonget(custConfigDataJson,newTableHeader,"json",json());
            inputJson = json();
            jsonput(inputJson,"modelJson",modelJson);
            jsonput(inputJson,"lvl1PriceTablesJson",lvl1PriceTablesJson);
            jsonput(inputJson,"customSKUsDataJson",customSKUsDataJson);
            jsonput(inputJson,"tierIndentationJson",tierIndentationJson);
            reponse = commerce.mergeCustomSKUDetails_c(inputJson);
            jsonremove(custConfigDataJson,newTableHeader);
            lvl1PriceTablesJson = jsonget(reponse,"lvl1PriceTablesJson","json",lvl1PriceTablesJson);
            modelJson = jsonget(reponse,"modelJson","json",modelJson);
            
            inputJson = json();
            jsonput(lvl1PriceTablesJson,"tableHeader",newTableHeader);
            jsonput(inputJson,"lvl1PriceTablesJson",lvl1PriceTablesJson);
	    jsonput(inputJson,"transactionPriceTableJson",transactionPriceTableJson); 
	    jsonput(inputJson,"modelJson",modelJson);
	    if(NOT isnull(jsonget(abstractModelsJson,jsonget(modelJson,"modelVarName","string","")))){
	        jsonput(inputJson,"abstractModel",jsonget(abstractModelsJson,jsonget(modelJson,"modelVarName","string","")));
	        jsonput(inputJson,"pseudoModelJson",pseudoModelJson);
	    } 
	    reponse = commerce.collectLvl1OptionClassDetails_c(inputJson);
	    lvl1PriceTablesJson = json();
	    modelJson = jsonget(reponse,"modelJson","json",modelJson);
	    pseudoModelJson = jsonget(reponse,"pseudoModelJson","json",pseudoModelJson);
	    transactionPriceTableJson = jsonget(reponse,"transactionPriceTableJson","json",transactionPriceTableJson);
        }
    }
    //###### Adding new Tables if Custom SKU Config table is not present in the Transaction ########
    
    inputJson = json();
    jsonput(inputJson,"modelJson",modelJson);
    jsonput(inputJson,"transactionPriceTableJson",transactionPriceTableJson);
    if(NOT isnull(jsonget(abstractModelsJson,jsonget(modelJson,"modelVarName","string","")))){
        jsonput(inputJson,"abstractModel",jsonget(abstractModelsJson,jsonget(modelJson,"modelVarName","string","")));
        jsonput(inputJson,"pseudoModelJson",pseudoModelJson);
    }
    response = commerce.collectModelDetails_c(inputJson);
    pseudoModelJson = jsonget(response,"pseudoModelJson","json",pseudoModelJson);
    transactionPriceTableJson = jsonget(response,"transactionPriceTableJson","json",transactionPriceTableJson);
    sbappend(retStr,jsonget(response,"retStr","string",""));
}

inputJson = json();
jsonput(inputJson,"pseudoModelJson",pseudoModelJson);
jsonput(inputJson,"transactionPriceTableJson",transactionPriceTableJson);
response = commerce.constructAbstractModels_c(inputJson);
transactionPriceTableJson = jsonget(response,"transactionPriceTableJson","json",transactionPriceTableJson);
sbappend(retStr,jsonget(response ,"retStr","string",""));    

inputJson = json();
jsonput(inputJson,"totalModels",totalModels);
jsonput(inputJson,"relationshipCreditsJson",relationshipCreditsJson);
jsonput(inputJson,"transactionPriceTableJson",transactionPriceTableJson);
response = commerce.constructTransactionSummary_c(inputJson);
//sbappend(retStr,jsonget(response,"retStr","string",""));
//################ Constructing Transaction Summary ##############################
//#################### Populating Assumption Data Json############################
abstractModelsArr = jsonkeys(abstractModelLineDataJson);
for eachModel in abstractModelsArr{
    jsonput(lineDataJson,eachModel,jsonget(abstractModelLineDataJson,eachModel,"json",json()));
}
jsonput(assumptionDataJson,"lineDataJson",lineDataJson);
jsonput(assumptionDataJson,"pseudoModels",join(jsonkeys(pseudoModelJson),"##"));
jsonput(assumptionDataJson,"abstractModelsDocJson",abstractModelsDocJson);
jsonput(assumptionDataJson,"activeSKUNumbers",sbtostring(activeSKUNumbers));
jsonput(assumptionDataJson,"activeSKUCount",activeSKUCountJson);
setattributevalue(1, "assumptionDataJson_t_c", jsontostr(assumptionDataJson));
//sbappend(retStr,"1~assumptionDataJson_t_c~",jsontostr(assumptionDataJson),"|");
//#################### Populating Assumption Data Json############################
return sbtostring(retStr);
