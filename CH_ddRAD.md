#CH samples

Data organisation and analyses

3 Nov 2015

Final demultiplexed and trimmed samples are organised in a folder on the GDC server: 

/gdc_home4/alexjvr/CHFinalSamples

Total: 1027 samples, 82 populations

Pop| n
:--:|:--:
abnd	|10
agra	|10
alpl|	10
apla|	20
arce|	20
bach|	10
bela|	10
bend|	20
berm|	10
bide|	20
birk|	9
bnnp|	10
boty|	10
buel|	10
cava|	10
csan|	20
csee|	20
egel|	18
ellw|	2
ente|	10
fess|	18
flor|	10
flue|	10
forn|	10
full|	20
fuor|	10
gdwe|	10
gola|	17
gott|	10
grma|	17
grsh|	10
gruu|	10
hdns|	10
jagg|	10
kand|	19
kebe|	10
laun|	10
lens|	19
lucm|	10
magn|	10
mart|	10
mctn|	10
meie|	20
mgns|	19
moal|	10
moir|	13
muet|	10
munt|	19
oalp|	10
orge|	20
otte|	10
pizo|	10
pozz|	9
prad|	7
rade|	10
roes|	10
rotc|	10
rusc|	10
sali|	10
saxm|	9
scai|	10
seeo|	9
seji|	10
shwe|	10
siec|	9
span|	10
star|	20
stba|	10
stir|	10
stls|	10
tals|	10
tamv|	20
tana|	20
trim|	19
trpa|	10
tsee|	18
vilt|	20
vora|	20
wale|	8
wdji|	10
wise|	9
zeni|	10


##CH530

Looking at the level of polymorphism according to canton and elevation. This is for a recommendation for Daniel Lee Jeffries from EPFL, who is sequencing the Rana temporaria genome. 

```
require(ggplot2)

###Heterozygosity plot

CH530.poly <- read.csv("pyRAD530.polyfreq_20160224.v2.csv")

CH530.poly$Site <- factor(CH530.poly$Site, levels=CH530.poly$Site)  #Makes Site and ordered factor, so that ggplot doesn't automatically sort the data before plotting

q <- qplot(Site, poly, fill=factor((Elev.Class), levels=c("Low","Mid","High")), data=CH530.poly, geom="boxplot", position="dodge") + scale_fill_manual(values=c("yellow","green","blue"))+theme_bw()+theme(legend.title=element_blank())

q + theme(axis.text.x=element_text(angle = 90, hjust = 0))

```

![alt_txt][npoly.CH530]
[npoly.CH530]:https://cloud.githubusercontent.com/assets/12142475/13290794/84bc9cf4-db15-11e5-97c9-2986a48ed4bb.png


