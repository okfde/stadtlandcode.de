---
layout: post
author: Marcel Belledin
title: Der Kultur-o-mat
snapshot: chart.png
---

Kultur-o-mat
------------

Unser Projekt gibt eine Orientierungshilfe in der Szenen- und Kulturlandschaft. Über einen kleinen Fragebogen erfährt man, wie man sich persönlich einschätzt und in welchen Kultur-Szenen man sich noch wohlfühlen könnte. Die Ergebnisse werden durch interaktive „Bubble Charts“ angezeigt. 
Für die Visualisierung benutzen wir D3, was für Data-Driven Documents steht und womit man relativ unkompliziert beeindruckendes aus den meist trockenen Zahlen zaubern kann.
Wer D3 noch nicht kennt, kann sich [hier](http://d3js.org/) umsehen. Es gibt dort auch einige Tutorials. 
Eine kleines Beispiel stelle ich gleich vor. Wer noch nie in Javascript programmiert hat, kann „guttenbergen“ bzw. den Code hier eintippen: [Tributary](http://enjalot.com/tributary/). Ich werde versuchen Dinge die mir wichtig erscheinen genauer zu erklären und hoffe, dass auch einige Neueinsteiger mit dem Code etwas anfangen können.

Falls Interesse da ist, kann sich jeder gerne überlegen, welche Daten aus dem Alltag gesammelt werden könnten, um als Grundlage zu dienen. Bis dahin nutzen wir zufällige Zahlen:

	function randomData() {
		return d3.range(10).map(function(i) {
			return {x: i / 9, y: Math.random()};
		});
	}

Mit der Funktion werden 10 mal x/y Werte generiert, die Punkte in einem Koordinatenkreuz darstellen. Da es uns erst mal darum geht irgendetwas anzuzeigen, bleibt die x- und y-Achse ohne Namen.

Was wir noch brauchen sind einige Werte die wir vorher festlegen:

	var MaxY = d3.max(randomData(), function(d) {return d.y;});
	var w = 400, //Breite - (w)idth
		h = 450, //Höhe - (h)eight
		m = 65, //Abstand - (m)argin
		x = d3.scale.linear().domain([0, 1]).range([0, w]),
		y = d3.scale.linear().domain([0, MaxY]).range([h, 0]);
	var color = d3.scale.category20c();

Zuerst ermitteln wir den maximalen Wert der y-Achse, da dies eine zufällig Zahl ist und wir dafür sorgen wollen, dass sie in dem Bereich angezeigt wird, den wir festlegen. Dafür geben wir an, wie groß das Fenster sein soll, in dem unsere Daten später angezeigt werden. 
Die Variablen x und y bedeuten, dass wir eine lineare Skala nutzen und unsere Ausgangsdaten (domain) in dem sichtbaren Bereich (range) zwischen dem 0 Pixel und der festgelegten Höhe und Breite anzeigen wollen. Dann nutzen wir noch die Farbpalette von d3, um später jedem Punkt eine eigene Farbe zuordnen zu können. 

Da wir auf [Tributary](http://enjalot.com/tributary/) nichts vorbereiten müssen, um d3 einzubinden, können wir einfach den bestehenden Platzhalter für unsere „skalierbare Vektorgrafik“ (svg) nutzen:

		var chart = d3.select("svg")
					  .append("svg")
					  .attr("width", w + m * 2)
					  .attr("height", h + m * 2);

Damit sagen wir, dass d3 den bestehenden Platzhalter für das „svg“ wählen soll und unsere „svg“ mit den angegebenen Werten für Höhe und Breite daran anhängen soll.
Fertig ist die Grundfläche für unser Diagramm (chart). Damit die fertige Visualisierung (vis) auch innerhalb des Fensters angezeigt wird, verschieben wir das ganze noch ein wenig:  

		var vis = chart.append("g")
	 				   .attr("transform", "translate(" + m + "," + m + ")");

Jetzt müssen wir nur noch die Daten anzeigen lassen:

		var node = vis.selectAll()  // wähle alles aus
			        .data(randomData()) // benutze diese Daten
			        .enter().append("path") // Hänge sie an die Visualisierung
			        .attr("transform", function(d) { return "translate(" + x(d.x) + "," + y(d.y) + ")"; })
			        .attr("d", d3.svg.symbol().type("triangle-down").size("800"))
			        .style({ 
			        	fill: function(d,i) { return color(i) }
			        })

Wir sagen, dass alles in der Visualisierung ausgewählt wird. Da noch keine Daten vorhanden sind, nutzen wir unsere zufälligen Daten. Die Befehlskette selectAll, data, enter und append wird am häufigsten in D3 benutzt, da sie die Grundlage dafür ist, dass die Daten auch angezeigt werden können. Wir müssen jetzt nur noch festlegen wo die Punkte im Diagramm angezeigt werden. D3 bietet auch eine Auswahl an Symbolen, die man Nutzen kann. Wir wählen das Dreieck und weißen im zum Schluss noch eine Farbe aus der zu beginn gewählten Palette zu. Fertig ist das Diagramm...

![Beispiel](/img/posts/chart.png)


Da die Stärken von d3 aber in der Animation liegen, machen wir direkt weiter und hängen diese Zeile an:

		.on("mousedown", function() {update(this);});

Wenn wir auf ein Dreieck klicken, soll etwas passieren: 

		update = function(thisObject){
					d3.select(thisObject)
					  .style("opacity",1) // legt die Deckkraft fest
		              .attr("d", d3.svg.symbol().type("triangle-down").size("500"))
		              .transition()
		                .duration(500)
		                .attr("d", d3.svg.symbol().type("triangle-down").size("1500")) //verändert die Größe
		                .style("opacity",0); // Deckkraft lässt Dreieck verschwinden
	              
		vis.selectAll() 
	       .data(randomData)
	       .enter().append("svg:path")
	       .style("fill", thisObject.style.fill)   // die gleiche Farbe
	       .attr("transform", function(d) { return "translate(" + x(d.x) + "," + y(d.y) + ")"; })
	       .attr("d", d3.svg.symbol().type("diamond").size("0")) //ein anderes Symbol
	       .style("opacity",0)
	       .transition()
	         .duration(300)
	         .attr("d", d3.svg.symbol().type("diamond").size("300"))
	         .style("opacity",0.72);
		}


Das neue in diesen Zeilen Code ist „transition“ und „duration“: Verwandle das Bestehende und brauche dafür soundsolange.

Hier nochmal alles zusammen und eine kleine Überraschung: Wenn man alles anklickt und mit der Maus jeden Diamanten berührt hat, verbreitet das Bild weihnachtliche Stimmung... (ok, mit etwas Fantasie...)

		function randomData() {
		    return d3.range(10).map(function(i) {
		        return {x: i / 9, y: Math.random()};
		    });
		}


		var MaxY = d3.max(randomData(), function(d) {return d.y;});

		var color = d3.scale.category20c();

		var w = 400, //Breite - (w)idth 
		    h = 450, //Höhe - (h)eight
		    m = 97, //Abstand - (ma)rgin
		    x = d3.scale.linear().domain([0, 1]).range([0, w]),
		    y = d3.scale.linear().domain([0, MaxY]).range([h, 0]);


		var chart = d3.select("svg")
		    .append("svg")
		    .attr("width", w + m * 2)
		    .attr("height", h + m * 2);

		var vis = chart.append("svg:g")
		    .attr("transform", "translate(" + m + "," + m + ")");


		var node = vis.selectAll()
		        .data(randomData())
		        .enter().append("path")
		        .attr("transform", function(d) { return "translate(" + x(d.x) + "," + y(d.y) + ")"; })
		        .attr("d", d3.svg.symbol().type("triangle-down").size("800"))
		        .style({ 
		         fill: function(d,i) { return color(i) }
		         })
		        .on("mousedown", function() {update(this);});


		update = function(thisObject){
		          d3.select(thisObject)
		            .style("opacity",1) // legt die Deckkraft fest
		            .attr("d", d3.svg.symbol().type("triangle-down").size("500"))
		            .transition()
		              .duration(500)
		              .attr("d", d3.svg.symbol().type("triangle-down").size("1500")) //verändert die Größe
		              .style("opacity",0); // Deckkraft lässt Dreieck verschwinden
		              
		           vis.selectAll()
		              .data(randomData)
		              .enter().append("svg:path")
		              .style("fill", thisObject.style.fill)
		              .attr("transform", function(d) { return "translate(" + x(d.x) + "," + y(d.y) + ")"; })
		              .attr("d", d3.svg.symbol().type("diamond").size("0"))
		              .on("mouseover", function(d,i) {
		                d3.select(this).transition().duration(300).style("fill","#000000"); })
		              .on("mouseout", function(d,i) {
		                d3.select(this).transition().duration(200).style("fill","white"); })
		              .style("opacity",0)
		              .transition()
		                .duration(640)
		                .attr("d", d3.svg.symbol().type("diamond").size("300"))
		                .style("opacity",0.72);
		}