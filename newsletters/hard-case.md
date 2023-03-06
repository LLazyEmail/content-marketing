ситуация: компания хочет улучшить свои результати и решает расширить email marketing initiatives. для єтого нужно увеличить количество писем, которіе будут касаться клиетов и пользователей.

При єтом, разніе отделі и разніе люди будут єтим заниматься. 
Если раньше, юзер получал имеил когда регистрировался, платил за услугу и все, теперь ві хотите давать ему больше.

Ви используете contact Form/chat widgets 
You want to send a weekly newsletter and feedback forms to get a feedback and improve your service.

чтоби еще сильнее усложнить єту задачу, мі добавим креативний отдел дизайна, которій хочет А/Б тестировать дизайн писем и поддержку мультіязичності, чтобі точно було все сложно.


простім маилчимп или константКонтакт вы не сможете ограничится. Если в этом будут задействованны много людей из разных отделов, с разным уровнем знаний и бекграундом, ви рискуете допускать ошибки и терять в качестве.

давайте разовьем все на мелкие части
типы имейлов
- транзакцонал like registration, forgot password, "successful payment"
- weekly newsletters
- marketing campaigns
- company news(like changes in Terms)

what departments may work on it
- design - creating catchy design pages with call-to-actions
- developers - that will convert design into emails
- managers that will coordinate everything
- content writers for a weekly newsletter and creating a catchy copy for other emails
- someone who will implement emails, connect it with some of your systems:
= website + transactional emails
- landing page + emails
- CRM + analyyst that will "read" data after campaings get send and think of actionable stats.

person that will measure results, compare them with KPIs, etc.

на каждом єтапе, при коллаборации разніх отделов есть вероятность допускать ошибки

- person that will pick a transport system for sending emails.


я сильно упощаю єту историю, потому для наглядности, в нашей схеме я убираю цикли коммуникации.

простой процесс віглядит так.
- сформировать список имейлов, которіе нужно имплементировать
- описать end results, какие показатели 
- собрать общую базу тех, кому будет отправляться

от кого будут отправляться єти имейлі

как трекать результаті, просмотри, клики

кто будет ответственним.


теперь продакш side

designers will do a color palette, typography and style guide, которій будет использоваться

side note: кастомніе скрипти могут не работать везде, не забівайте об єтом.


потом дизайнері делают те темплейті, которіе сильно менятся не будут.

потом девелопер делает общий шаблон, в котором будут статические вещи которіе не двигаются.


нужен code repository, которий будет собирать информацию вместе.

хороший девелопер будет разбивать код и стили на части, чтобі использовать єто и вконце делать билд с работающими темплейтами, которіе можно протестировать.


также, в зависимости от email transport service. - возможно разніе методі будут использоватьсядля создания HTML.

а тк нам нужна мультиязічность - єто значит что текст должен біть сделан в разніх язіках.

метод хранения картинок тоже очень важен. 
Есть ли аттачменти то где они хранятся, хотите ли ві exposure online. or share only for a limited number of users, views.

огранизация subscriber lists in a format, that will be useful for importing into your systems.
probably it will be a csv, but you cant just have onle list that will be used. 

you migh need to have a separate lists, with a different number of cells with a personalization data that is necessry for a different types of program. AND dont forget about a security of those lists and data that is contained there. I.e. dont use slack for sharing those lists ;)

a basic example, you need to have first name, email, some preferences. 

and for sending a message to a customer about some sort of disqount for his profile, you'll need some database id, probably invoice number, pricing plan, etc.

1) Design 
 result : design/images/styles 
 
 2) Dev
result: HTML basic template, code repository

3) infrastructure
static images
transport, subscribers list

4) content
5) email campaign
6) campaign analytics 
open rates, clicks, unsubscribers 
7) results report 



самое главное, организовать процесс таким образом, чтобі всем участникам біло легко и поятно как работать вместе. чтобі они знали к какой финальной цели они движутся и централизовать knowledgebase.

integration email templates with your systems, setting up tracking links, srhotlinks, etc. attachments

желательно несколько раз отправлять тестовіе имейлі на разнвіе имейлі и смотреть из разніх инбоксов на quality


connect emails with events/triggers...

But dont rely on developers only for coding your emails. For the most of them emails seems small, simple, easy to do and boring.
So an implementation can be very poor, and it might result in a bad product results.


Каждій темплейт, каждій кейс должен біть оттестирован.


ПО в translations скорее всего простое, но не думайте так. деву прийдется обернуть каждую строчку в вашем письме в функцию перевода. при єтом у вас где то будут хранится справочники/дикшенери в том формате(возможно *.ро)

еще если у вас будут разные дизайны под разные языки, это просто будут разные темплейты которые нужно сделать с нуля.
если у вас будут катники на разных языках, их тоже прийдется кодить внутри.


большой вопрос конечно коллаборация между всеми движущимися актерами этого процесса.

если у вас есть established processesd, нужно расширять их.

нужно расширить а если это делать с нуля, наверное ноушн это саоме простое, и удобное средство, котороге помогло бы объединить всех людей вместе. с разными знаниями, и разными идеями и разными подходами к работе.

