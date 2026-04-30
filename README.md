# temp-code


retStr = stringbuilder();
function = jsonget(inputJson,"function","string","");
allModelsJson = json();
if(allModels_t_c <> ""){
	allModelsJson = json(allModels_t_c);
}
modelKeys = jsonkeys(allModelsJson);
adHocModelDetailsJson = json();
productTypeDetailsJson = json();
coreAssocVarName = "";
coreModelJson = json();
coreModelVarName = "";
totalNonCoreProducts = 0;
if(coreAssociation_t_c <> "N" AND coreAssociation_t_c <> ""){
	coreAssocVarName = coreAssociation_t_c;
}

results = bmql("Select ModelVarName,ProductType,CoreAssocVarName,PackageSKU from CoreProducts where ModelVarName IN $modelKeys OR CoreAssocVarName = $coreAssociation_t_c");
for each in results {
	modelDetailsJson = json();
	jsonput(modelDetailsJson,"productType",get(each,"ProductType"));
	jsonput(modelDetailsJson,"coreAssocVarName",get(each,"CoreAssocVarName"));
	if(get(each,"ProductType") == "Core"){
		coreAssocVarName = get(each,"CoreAssocVarName");
		jsonput(coreModelJson,coreAssocVarName,get(each,"PackageSKU"));
		coreModelVarName = get(each,"ModelVarName");
	}
	else{
		totalNonCoreProducts = totalNonCoreProducts+1;
	}
	jsonput(productTypeDetailsJson,get(each,"ModelVarName"),modelDetailsJson);
}

//-------------- Function to set Ad Hoc Packages Array during Boolean Auto Trigger------------------------------------
if(function == "adHocPackages_t_c"){
	adHocPackageJsonArr = jsonarray();
	for each in modelKeys{
		currentModelJson = jsonget(allModelsJson,each,"json",json());
		productTypeJson = jsonget(productTypeDetailsJson,each,"json",json());
		if(jsonget(productTypeJson,"productType","string","") == "Non Core"){
			adHocPackageJson = json();
			jsonput(adHocPackageJson,"ahpArr_model_t_c",jsonget(currentModelJson,"modelName","string",""));
			jsonput(adHocPackageJson,"ahpArr_modelVariableName_t_c",each);
			if(NOT adHocPackages_t_c){
				jsonput(adHocPackageJson,"ahpArr_applyBundling_t_c",adHocPackages_t_c);
			}
			jsonarrayappend(adHocPackageJsonArr,adHocPackageJson);
		}
		if(NOT adHocPackages_t_c){
			jsonput(adHocModelDetailsJson,each,adHocPackages_t_c);
		}
	}
	sbappend(retStr, "1~adHocPackagesArray_t_c~",jsonarraytostr(adHocPackageJsonArr),"|");
}
//-------------- Function to set Ad Hoc Packages Array during Boolean Auto Trigger------------------------------------

//-------------- Function to set Ad Hoc Packages Array during Update Action ------------------------------------
if(function == "updateAdHocPackages_t_c"){
	indices = range(jsonarraysize(adHocPackagesArray_t_c));
	for index in indices{
		currentModelDetailsJson = jsonarrayget(adHocPackagesArray_t_c,index,"json");
		jsonput(adHocModelDetailsJson,jsonget(currentModelDetailsJson ,"ahpArr_modelVariableName_t_c","string",""),jsonget(currentModelDetailsJson,"ahpArr_applyBundling_t_c","boolean",false));
	}
	//Bibin Sajeevan : 16 June 2025 : Calling Pricing Lib
	reqJson = json();
	jsonput(reqJson, "adHocModelDetailsJson", adHocModelDetailsJson);
	jsonput(reqJson, "InvokeAction", "Initialization");
	sbappend(retStr,commerce.pricingLibrary_c(jsontostr(reqJson)));
	//Bibin Sajeevan : 16 June 2025 : Calling Pricing Lib
}
//-------------- Function to set Ad Hoc Packages Array during Update Action --------------------------------------

//-------------- Function to set Ad Hoc Packages Array during Configuration ------------------------------------
if(function == "configure" OR function == "_remove_transactionLine"){
	indices = range(jsonarraysize(adHocPackagesArray_t_c));
	adHocPackageJsonArr = jsonarray();
	existingAdHocProducts = json();
	for index in indices{
		currentModelDetailsJson = jsonarrayget(adHocPackagesArray_t_c,index,"json");
		modelVarName = jsonget(currentModelDetailsJson ,"ahpArr_modelVariableName_t_c","string","");
		if(findinarray(modelKeys,modelVarName) <> -1){
			jsonput(existingAdHocProducts,jsonget(currentModelDetailsJson ,"ahpArr_modelVariableName_t_c","string",""),jsonget(currentModelDetailsJson,"ahpArr_applyBundling_t_c","boolean",false));
			jsonarrayappend(adHocPackageJsonArr,currentModelDetailsJson);
		}
	}
	for each in modelKeys{
		if(isnull(jsonget(existingAdHocProducts,each))){
			currentModelJson = jsonget(allModelsJson,each,"json",json());
			productTypeJson = jsonget(productTypeDetailsJson,each,"json",json());
			if(jsonget(productTypeJson,"productType","string","") == "Non Core"){
				adHocPackageJson = json();
				jsonput(adHocPackageJson,"ahpArr_model_t_c",jsonget(currentModelJson,"modelName","string",""));
				jsonput(adHocPackageJson,"ahpArr_modelVariableName_t_c",each);
				//jsonput(adHocPackageJson,"ahpArr_applyBundling_t_c",true);
				jsonarrayappend(adHocPackageJsonArr,adHocPackageJson);
				//jsonput(adHocModelDetailsJson,each,false);
			}
		}
	}
	sbappend(retStr, "1~adHocPackagesArray_t_c~",jsonarraytostr(adHocPackageJsonArr),"|");
}
//-------------- Function to set Ad Hoc Packages Array during Configuration --------------------------------------

//-------------- Setting Line Item Attributes ---------------------------------

for line in transactionLine{
	lineModelDetailsJson = json();
	if(line.modelDetails_l_c <> ""){
		lineModelDetailsJson = json(line.modelDetails_l_c);
	}
	modelVarName = jsonget(lineModelDetailsJson,"modelVarName","string","");
	if(isnull(modelVarName)){
		modelVarName = "";
	}
	
	if(NOT isnull(jsonget(adHocModelDetailsJson,modelVarName))){
		if(lower(line.feeType_l_c) == "implementation" OR lower(line.feeType_l_c) == "per occurrence" OR lower(line.feeType_l_c) == "monthly" OR lower(line.feeType_l_c) == "license"){
			setBool = jsonget(adHocModelDetailsJson,modelVarName,"boolean",false);
			sbappend(retStr,line._document_number,"~includedOrWaived_l_c~",string(setBool),"|");
			if(setBool){
				sbappend(retStr,line._document_number,"~packagedWith_l_c~",jsonget(coreModelJson,coreAssocVarName,"string",""),"|");
			}
			else{
				sbappend(retStr,line._document_number,"~packagedWith_l_c~|");
			}
		}
	}
	//elif(modelVarName == coreModelVarName AND totalNonCoreProducts > 0){
	//	sbappend(retStr,line._document_number,"~includedOrWaived_l_c~false|");
	//	sbappend(retStr,line._document_number,"~packagedWith_l_c~|");
	//}
	if(function == "includedOrWaived_l_c" OR function == "configure" OR function == "_remove_transactionLine"){
		if(line.includedOrWaived_l_c AND modelVarName <> coreModelVarName){
			sbappend(retStr,line._document_number,"~packagedWith_l_c~",jsonget(coreModelJson,coreAssocVarName,"string",""),"|");
		}
		else{
			sbappend(retStr,line._document_number,"~packagedWith_l_c~|");
		}
	}
	if(line.displayNetPrice_l_c == "Quote" AND line.updatedListPrice_l_c == 0.0){
		sbappend(retStr,line._document_number,"~includedOrWaived_l_c~false|");
		sbappend(retStr,line._document_number,"~packagedWith_l_c~|");
	}
}
//-------------- Setting Line Item Attributes ---------------------------------
return sbtostring(retStr);
