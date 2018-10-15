# JQuery 更新表格序号和克隆

标签（空格分隔）： JQuery

---
    1.克隆
    获取某个元素的Id，clone()克隆
    var childRule = $("#tScheChangeLeaveSetting").clone();


    2.更新序号
    克隆后找到input标签，foreach遍历，（获取当前元素的name和id）
    	childRule.find("input").each(function(){
			$(this).attr("name","childRule[" + index + "]." + this.name)
			$(this).attr("id","childRule" + index + "_" + this.id)
			});
		childRule.find("select").each(function(){
			$(this).attr("name","childRule[" + index + "]." + this.name)
			$(this).attr("id","childRule" + index + "_" + this.id)
			});
			
			
	3.完整实例
	var index = 0;
	function addChildRule(){
		var childRule = $("#tScheChangeLeaveSetting").clone();
		childRule.attr("id",childRule.attr("id").substr(0,21) + index);
		//var childRuleFieldset = $("#childRuleDiv").find("fieldset").clone();
		//childRule.appendTo(childRuleFieldset);
		childRule.find("tr :first").show();
		childRule.find("input").each(function(){
			$(this).attr("name","childRule[" + index + "]." + this.name)
			$(this).attr("id","childRule" + index + "_" + this.id)
			});
		childRule.find("select").each(function(){
			$(this).attr("name","childRule[" + index + "]." + this.name)
			$(this).attr("id","childRule" + index + "_" + this.id)
			});
		$("#tScheChangeLeaveSetting").parent().append(childRule);
		index++;
	}
			
	




