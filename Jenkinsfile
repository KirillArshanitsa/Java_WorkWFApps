import java.time.LocalTime;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import groovy.json.JsonSlurper;

def XmlConfFile = "Config.xml"
def EmailAuthor = "email."

def runGetRequestWithoutChoice(monitoringHost, partOfMonitoringRequest, moduleName){
	def monitoringURL = "http://" + monitoringHost + partOfMonitoringRequest + moduleName
	println("URL запроса - " + monitoringURL)
	def url = new URL(monitoringURL)
	def connection = url.openConnection()
	connection.requestMethod = 'GET'
	def hostAppMap = new HashMap();
	def diffAppNames = new HashSet();
	if (connection.responseCode == 200) {;
	  def jsonList = connection.content.text
	  if (jsonList.equals("[]") | jsonList.equals("")) {
		  	ansiColor('xterm') {
				echo "\u001B[31mОшибка - запрос к мониторингу ${monitoringURL} вернул пустое значение\u001B[m"
			}	
			println(jsonList);
			sh "exit 1"
	  }
	  jsonList = jsonList.substring(1, jsonList.length() - 1);
	  def jsonSlurper = new JsonSlurper()
	  for (String json : jsonList.split("}},")) {
		 json = json + "}}";
		 obj = jsonSlurper.parseText(json)
		 hostAppMap.put(obj.host+":"+ obj.port, obj.module.name)
		}
	  }
	else {
		ansiColor('xterm') {
			echo "\u001B[31mПри запросе ${monitoringURL} вернулся код ответа не равный 200\u001B[m"
		}			
		println(connection.responseCode)
		sh "exit 1"
	}
	//TODO поменять проверку на не совпадение ответа module":{"name"} с хостов при запросе getHostData?s=имя модуля
	for (val in hostAppMap.values()){
		diffAppNames.add(val)
	}	
	if (diffAppNames.size() > 1){
		ansiColor('xterm') {
			echo "\u001B[31mПри запросе ${monitoringURL} вернулись разные значения module:name \u001B[m"
			println("Хост и приложение")
			println(hostAppMap.entrySet())
			sh "exit 1"
		}			
	}
	return hostAppMap
}

def runGetRequestWithChoice(monitoringHost, partOfMonitoringRequest, moduleName){
	def monitoringURL = "http://" + monitoringHost + partOfMonitoringRequest + moduleName
	println("URL запроса - " + monitoringURL)
	def url = new URL(monitoringURL)
	def connection = url.openConnection()
	connection.requestMethod = 'GET'
	def hostAppMap = new HashMap();
	def diffAppNames = new HashSet();
	def colorGreen = "<b style=\'color:#008000\'>"
	def colorRed = "<b style=\'color:#ff0000\'>"
	if (connection.responseCode == 200) {
	  def textColor
	  def jsonList = connection.content.text
	  if (jsonList.equals("[]") | jsonList.equals("")) {
		  	ansiColor('xterm') {
				echo "\u001B[31mОшибка - запрос к мониторингу ${monitoringURL} вернул пустое значение\u001B[m"
			}	
			println(jsonList);
			sh "exit 1"
	  }
	  //TODO поменять
	  jsonList = jsonList.substring(1, jsonList.length() - 1);
	  def jsonSlurper = new JsonSlurper()
	  for (String json : jsonList.split("}},")) {
		 json = json + "}}";
		 obj = jsonSlurper.parseText(json)
		 if (obj.module.name.equalsIgnoreCase(moduleName)){			 
			 if (obj.module.status =='OK'){
				 textColor=colorGreen
			 }
			else {textColor=colorRed}
			str = '"'+obj.host+":"+ obj.port + ';  <em> Группа мониторинга - </em>' + obj.group + ' <em> Cтатус приложения </em>' + moduleName + ' - ' + textColor + obj.module.status +'</b> <em> Версия - </em>' + obj.module.version + '"'
			str = str.replaceAll(",","")				
			hostAppMap.put(str, obj.module.name)
			}
		}
	  }
	else {
		ansiColor('xterm') {
			echo "\u001B[31mПри запросе ${monitoringURL} вернулся код ответа не равный 200\u001B[m"
		}	
		println(connection.responseCode)
		sh "exit 1"
	}
	if (hostAppMap.size() == 0){
		ansiColor('xterm') {
			echo "\u001B[31mПри запросе ${monitoringURL} не найден модуль с именем ${moduleName} \u001B[m"
			println("Хост и приложение")
			println(hostAppMap.entrySet())
			sh "exit 1"
		}			
	}	

	//TODO поменять проверку на не совпадение ответа module":{"name"} с хостов при запросе getHostData?s=имя модуля
	// for (val in hostAppMap.values()){
		// diffAppNames.add(val)
	// }	
	// if (diffAppNames.size() > 1){
		// ansiColor('xterm') {
			// echo "\u001B[31mПри запросе ${monitoringURL} вернулись разные значения module:name \u001B[m"
			// println("Хост и приложение")
			// println(hostAppMap.entrySet())
			// sh "exit 1"
		// }			
	// }
	return hostAppMap
}	


def parseXmlConfFile(xmlConfFile, moduleName, standName){
	def readConfigFile = readFile(xmlConfFile)
	def moduleUsers
	def moduleTechWindow
	def monitoringHost
	HashSet allFindedModules = new HashSet();
	HashSet allFindedStand = new HashSet();
	def findedModule
	def list = new XmlSlurper().parseText(readConfigFile)
	def partOfRequestMonitoringHost = list.MonitoringInfo.partOfRequest.toString()
	def goupAdminEmail = list.AdminUsers.GroupAdminEmail.toString()
	list.MonitoringInfo.Stand.each{ stand ->
		if (stand['@Name'].toString().equalsIgnoreCase(standName)){
			monitoringHost = stand.Host
		};
		allFindedStand.add(stand['@Name'].toString().trim());
	}		
	list.Module.each{ module ->
	   if (module['@Name'].toString().contains(",")){
		  tempModulesList = module['@Name'].toString().split(",");
		  tempModulesList.each{tempModule ->
			tempModule = tempModule.trim();
			allFindedModules.add(tempModule);
			if (tempModule.equalsIgnoreCase(moduleName)){
			  findedModule = tempModule;
		  	  module.Stand.each{
					stand -> 
					if (stand['@Name'].toString().equalsIgnoreCase(standName)){
						moduleUsers=stand.Users.toString();
						moduleTechWindow=stand.TechWindow.toString();
					}
			  }	
			}
		  }
	  }
	  else{
		allFindedModules.add(module['@Name'].toString().trim());  
	  }
	  if (module['@Name'].toString().equalsIgnoreCase(moduleName)){
		findedModule = moduleName;
		module.Stand.each{
			stand -> if (stand['@Name'].toString().equalsIgnoreCase(standName)){
				moduleUsers=stand.Users.toString();
				moduleTechWindow=stand.TechWindow.toString();
			}
		}
	  }
	}	
	moduleUsers = moduleUsers + "," + list.AdminUsers.Users.toString();
	if (!findedModule){
		ansiColor('xterm') {
			echo "\u001B[31mВ конф. файле не найдено модуля с именем - ${moduleName}\u001B[m"
		}			
		sh "exit 1"		
	}	
	if (!moduleUsers){
		ansiColor('xterm') {
			echo "\u001B[31mНе найдено ни одного пользователя, которому разрешено работать с модулем - ${moduleName} на стенде ${standName}\u001B[m"
		}			
		sh "exit 1"
	}
	if (!moduleTechWindow){
		ansiColor('xterm') {
			echo "\u001B[31mНе найдено ни одного тех окна для модуля - ${moduleName} на стенде ${standName}\u001B[m"
		}		
		sh "exit 1"
	}
	if (!monitoringHost){
		ansiColor('xterm') {
			echo "\u001B[31mНе найдено ни одного сервера мониторинга для стенда - ${moduleName}\u001B[m"
		}			
		sh "exit 1"
	}
	if (!partOfRequestMonitoringHost){
		ansiColor('xterm') {
			echo "\u001B[31mВ конф файле не найдено значение partOfRequest для отправки запроса к сереру мониторинга.\u001B[m"
		}		
		sh "exit 1"		
	}
	moduleTechWindow = moduleTechWindow.replaceAll(" ","")
	List tempList = new ArrayList(allFindedModules)
	Collections.sort(tempList);
	return [moduleUsers, moduleTechWindow, tempList, allFindedStand, monitoringHost, partOfRequestMonitoringHost, goupAdminEmail]
}


def checkUserPermission(userName, moduleName, moduleUsers){
	println("Оригинальное имя пользователя в Jenkins - " + userName)
	moduleUsers = moduleUsers.toLowerCase().replaceAll(" ","")
	userName = userName.toLowerCase()
	if (moduleUsers.contains(",")){
		if (moduleUsers.split(",").contains(userName)){
			println("Пользователю " + userName + " разрешено работать с приложением - " + moduleName)
		}
		else {
			ansiColor('xterm') {
				echo "\u001B[31mПользователю ${userName} не разрешено работать с приложением ${moduleName} на стенде ${env.StandName} !\u001B[m"
			}				
			sh "exit 1"
		}	
	}	
	else{
		if (moduleUsers.equals(userName)){
			println("Пользователю " + userName + " разрешено работать с приложением - " + moduleName)
		}
		else {
			ansiColor('xterm') {
				echo "\u001B[31mПользователю ${userName} не разрешено работать с приложением ${moduleName} на стенде ${env.StandName} !\u001B[m"
			}			
			sh "exit 1"
		}
	}	
}		

def checkTechWindowTime(moduleTechWindow, moduleName, standName){
	if (moduleTechWindow.equalsIgnoreCase("None")){	
		println("У модуля ${moduleName} не задано тех окно на стенде ${standName}")		
		return true
	}	
	def myTimeStr = LocalTime.now().toString()
	def currnetTimeStr = myTimeStr.substring(0, myTimeStr.lastIndexOf(":")).replaceAll(":","")
	println("Обнаруженное время тех окна для модуля " + moduleName + " на стенде " + standName + " - " + moduleTechWindow)
	println("Проверка Тех.окна, время запуска - " + myTimeStr)
	if (moduleTechWindow.contains(",")){
		def listReuslTechWindowCheck = []
		moduleTechWindow.split(",").each{
			it -> isTechWindow = null;
			BeginTechWindow = it.substring(0, it.indexOf("-")).replaceAll(":",""); EndMTechWindow = it.substring(it.indexOf("-") + 1, it.length()).replaceAll(":","");
			if (BeginTechWindow < EndMTechWindow){
				if (BeginTechWindow < currnetTimeStr & currnetTimeStr < EndMTechWindow){
					isTechWindow = true
				}
				else {
					isTechWindow = false
				}
			}
			else{
				if (BeginTechWindow < currnetTimeStr | currnetTimeStr < EndMTechWindow){
					isTechWindow = true
				}
				else {
					isTechWindow = false
				}
			}	
			listReuslTechWindowCheck.add(isTechWindow);
		}
		if (listReuslTechWindowCheck.contains(true)){
			return true
		}
		else return false
	}
	else{
		BeginTechWindow = moduleTechWindow.substring(0, moduleTechWindow.indexOf("-")).replaceAll(":",""); 
		EndMTechWindow = moduleTechWindow.substring(moduleTechWindow.indexOf("-") + 1, moduleTechWindow.length()).replaceAll(":","");
		if (BeginTechWindow < EndMTechWindow){
			if (BeginTechWindow < currnetTimeStr & currnetTimeStr < EndMTechWindow){
				return true
			}
			else {
				return false
			}
		}
		else{
			if (BeginTechWindow < currnetTimeStr | currnetTimeStr < EndMTechWindow){
				return true
			}
			else {
				return false
			}
		}	
	}	
}

def workHostResult(hostListString){
	hostList = hostListString.split(',')
	for (val=0; hostList.size()> val; val = val +1){
		hostList[val] = hostList[val].substring(0, hostList[val].indexOf(";"))
	}	
	return hostList.toString()
}

	
node('masterLin'){
	def CurrentUser = currentBuild.rawBuild.getCause(Cause.UserIdCause).getUserId()
	def GroupAdminEmail
	try{
		git'https://gitUrl.git'
		stage('Выполнение'){
			def infXmlConfFile = parseXmlConfFile(XmlConfFile, env.ModuleName, env.StandName)
			def allStandName = new ArrayList(infXmlConfFile[3])
			Collections.sort(allStandName);
			def allModuleName = new ArrayList(infXmlConfFile[2])
			def monitoringHost = infXmlConfFile[4]
			def partOfMonitoringRequest = infXmlConfFile[5]
			GroupAdminEmail = infXmlConfFile[6]
			def resultModuleName

			properties([parameters([
					choice(choices: allStandName, description: 'Имя стенда', name: 'StandName'),
					choice(choices: 'Restart\nStop\nStart\n', description: 'Действие', name: 'Action'),
					choice(choices: '10\n5\n15\n20\n25\n', description: 'Время ожидания в мин', name: 'Timeout'),
					choice(choices: allModuleName, description: "<p>Имя модуля для поиска серверов.</p>", name: 'ModuleName'),
					//choice(choices: 'Yes\nNo\n', description: 'Работа с модулями будет идти последовательно или параллельно. При параллельной работе действия выполнятся на всех серверах сразу. ', name: 'Parallel'),
					choice(choices: 'No\nYes\n', description: 'Работа со всеми найденными серверами или предоставление возможности выборана каких серверах нужно работать. No - Выбора серверов нет. Yes - Выбор серверов есть.', name: 'ChoiceServer'),]),
					pipelineTriggers([])])		
			
			def ModuleUsers = infXmlConfFile[0]
			def ModuleTechWindow = infXmlConfFile[1]
			checkUserPermission(CurrentUser, env.ModuleName, ModuleUsers)		
			def resultCheckTechWindTime = checkTechWindowTime(ModuleTechWindow, env.ModuleName, StandName)
			if (resultCheckTechWindTime){
					println("Вермя запуска - " + LocalTime.now().toString())
			}
			else {
				timeout(time: 180, unit: 'SECONDS') { 
					input message: 'Запуск выполняется НЕ в тех окно!! \n Для продолжения необходимо явно подтвердить, что согласование работ получено.',
					parameters: [[$class: 'ChoiceParameter', choiceType: 'PT_CHECKBOX', description: '', filterLength: 1, filterable: false, name: 'Подтверждение наличая согласования.', randomName: 'choice-parameter-392186498983127']]
					echo "Подтверждение о наличае согласования на проведение работ получено от " + CurrentUser	
				}
			}	
			def serverList
			def srvList		
			if (env.ChoiceServer.equalsIgnoreCase("Yes")){
				def mapHostApp = runGetRequestWithChoice(monitoringHost, partOfMonitoringRequest, env.ModuleName)
				timeout(time: 180, unit: 'SECONDS') { 
					serverList = input message: 'Выбор серверов', parameters: [
						[$class: 'CascadeChoiceParameter', choiceType: 'PT_CHECKBOX', description: 
						'Приложение ' + env.ModuleName +' обнаружено на серверах стенда ' + env.StandName +', перечисленных выше. Выберите с какими работать.', filterLength: 1, 
						 filterable: false, name: '', randomName: 'choice-parameter-1170008735291079', referencedParameters: '', 
						 script: [$class: 'GroovyScript', fallbackScript: [classpath: [], sandbox: true, script: ''], 
						 script: [classpath: [], sandbox: true, script: mapHostApp.keySet().toString()]]]
					]
				}	
				if (!serverList){
					ansiColor('xterm') {
						echo "\u001B[31mОшибка - вы не указали ни одного сервера для работы, на этапе выбора серверов.\u001B[m"
					}					
					sh "exit 1"
				}
				resultModuleName = mapHostApp.values()[0]			
				srvList = workHostResult(serverList).replaceAll("\\[","\"").replaceAll("]","\"").replaceAll(" ","")			
			}
			else{			
				serverList = runGetRequestWithoutChoice(monitoringHost, partOfMonitoringRequest, env.ModuleName)								
				resultModuleName = serverList.values()[0]
				srvList = serverList.keySet().toString().replaceAll("\\[","\"").replaceAll("]","\"").replaceAll(" ","")	
			}
			println("Выбранные сервера: " + srvList)
			def Parallel = "YES"
			withCredentials([usernamePassword(credentialsId: 'BH_WF', passwordVariable: 'pwd', usernameVariable: 'username')]) {
				def executedScript = "java -jar WorkWFModuleHttp.jar " + env.StandName  + " " + env.Action + " " + resultModuleName + " " + srvList + " " + username + " " + pwd + " " + env.Timeout + " " + Parallel//env.Parallel
				echo "Выполняется команда - " + executedScript
				sh (script: executedScript, returnState: true)
			}
		}		
	} catch (e){
		println("Отправка email о результате ошибки.")
		emailext attachLog: true, body: "<p><span style=\'color:#ff0000\'><b>Не успешно</b></span> выполен job - ${env.Action} модуля ${env.ModuleName} на стенде ${env.StandName}, запущенный пользователем ${CurrentUser}<br>URL задания - ${env.BUILD_URL} </p>", compressLog: true, mimeType: 'text/html', subject: 'Job Work Wf apps FAILURE', to: "${GroupAdminEmail} ; ${EmailAuthor}"
		ansiColor('xterm') {		
			echo "Ошибка - \u001B[31m${e}\u001B[m"
		}
		error e.toString()
	} 
	buildStatus = currentBuild.currentResult
	println("Job status - " + buildStatus)
	println("Отправка email о результате.")
	emailext attachLog: true, body: "<p> Job - ${env.Action} модуля ${env.ModuleName} на стенде ${env.StandName}, запущенный пользователем ${CurrentUser} завершился - ${buildStatus}<br>URL задания - ${env.BUILD_URL} </p>", compressLog: true, mimeType: 'text/html', subject: "Job Work Wf apps ${buildStatus}", to: "${GroupAdminEmail} ; ${EmailAuthor}"
}

