# otp_demo_panel.py
# SAFE DEMO/TEST BOT
# Generates local random test codes only.
# It does NOT connect to SMS/OTP panels, phone numbers, or verification services.

import os
import asyncio
import random
import logging
import re
import json
import threading
from flask import Flask
from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup, ReplyKeyboardMarkup, CopyTextButton
from telegram.constants import ParseMode
from telegram.ext import Application, CommandHandler, MessageHandler, CallbackQueryHandler, ContextTypes, filters

# ===================== FLASK KEEP-ALIVE =====================
flask_app = Flask(__name__)

@flask_app.route('/')
def index():
    return "✅ OTP Panel Bot is running!", 200

def run_flask():
    flask_app.run(host='0.0.0.0', port=int(os.environ.get('PORT', 8080)))

BOT_TOKEN = "8764978166:AAF4kcucI1unskbmtDDImyEAtmRV_f9Pq-I"
ADMIN_ID = 6136815573
GROUP_ID = -1003875639913

OTP_LENGTH = 6
PAGE_SIZE = 12
POST_INTERVAL = 60  # seconds; admin can change with /setinterval
AUTO_POSTING = False

COUNTRIES = [
    ('(1)(US)🇺🇸 United States', '🇺🇸', '5913463998522592692'),
    ('(380)(UA)🇺🇦 Ukraine', '🇺🇦', '5911406692007941050'),
    ('(48)(PL)🇵🇱 Poland', '🇵🇱', '5913550391789752571'),
    ('(7)(KZ)🇰🇿 Kazakhstan', '🇰🇿', '5913724621433082323'),
    ('(86)(CN)🇨🇳 China', '🇨🇳', '5913779335021466780'),
    ('(994)(AZ)🇦🇿 Azerbaijan', '🇦🇿', '5911197578640233518'),
    ('(?)(EU)🇪🇺 European Union', '🇪🇺', '5911106310585193018'),
    ('(374)(AM)🇦🇲 Armenia', '🇦🇲', '5913272455866093666'),
    ('(79)(RU)🇷🇺 Russian Federation', '🇷🇺', '5913274246867456342'),
    ('(998)(UZ)🇺🇿 Uzbekistan', '🇺🇿', '5911051846104912282'),
    ('(49)(DE)🇩🇪 Germany', '🇩🇪', '5911096835887337583'),
    ('(81)(JP)🇯🇵 Japan', '🇯🇵', '5913293711659241040'),
    ('(90)(TR)🇹🇷 Turkey', '🇹🇷', '5910995113881901195'),
    ('(375)(BY)🇧🇾 Belarus', '🇧🇾', '5911011185649521599'),
    ('(44)(GB)🇬🇧 United Kingdom', '🇬🇧', '5913443365499703513'),
    ('(91)(IN)🇮🇳 India', '🇮🇳', '5913754823643107921'),
    ('(55)(BR)🇧🇷 Brazil', '🇧🇷', '5911148568768418614'),
    ('(260)(ZM)🇿🇲 Zambia', '🇿🇲', '5913564754160389778'),
    ('(967)(YE)🇾🇪 Yemen', '🇾🇪', '5913346492512341993'),
    ('(84)(VN)🇻🇳 Vietnam', '🇻🇳', '5913428887164949581'),
    ('(379)(VA)🇻🇦 Holy See', '🇻🇦', '5911211932420938860'),
    ('(678)(VU)🇻🇺 Vanuatu', '🇻🇺', '5913511535220625585'),
    ('(598)(UY)🇺🇾 Uruguay', '🇺🇾', '5913623088406204470'),
    ('(971)(AE)🇦🇪 United Arab Emirates', '🇦🇪', '5913726554168365343'),
    ('(256)(UG)🇺🇬 Uganda', '🇺🇬', '5913488939397681980'),
    ('(993)(TM)🇹🇲 Turkmenistan', '🇹🇲', '5913315521503170180'),
    ('(216)(TN)🇹🇳 Tunisia', '🇹🇳', '5911332947419468671'),
    ('(228)(TG)🇹🇬 Togo', '🇹🇬', '5913423260757790970'),
    ('(66)(TH)🇹🇭 Thailand', '🇹🇭', '5913617968805187987'),
    ('(255)(TZ)🇹🇿 Tanzania', '🇹🇿', '5911418949844603556'),
    ('(992)(TJ)🇹🇯 Tajikistan', '🇹🇯', '5911287639809463107'),
    ('(41)(CH)🇨🇭 Switzerland', '🇨🇭', '5913271227505448072'),
    ('(46)(SE)🇸🇪 Sweden', '🇸🇪', '5911156510162949403'),
    ('(268)(SZ)🇸🇿 Eswatini', '🇸🇿', '5913374525763883286'),
    ('(597)(SR)🇸🇷 Suriname', '🇸🇷', '5913275539652611719'),
    ('(249)(SD)🇸🇩 Sudan', '🇸🇩', '5911387497799094470'),
    ('(34)(ES)🇪🇸 Spain', '🇪🇸', '5911193287967904547'),
    ('(94)(LK)🇱🇰 Sri Lanka', '🇱🇰', '5911293163137406640'),
    ('(211)(SS)🇸🇸 South Sudan', '🇸🇸', '5911406262511211744'),
    ('(27)(ZA)🇿🇦 South Africa', '🇿🇦', '5911203119148044594'),
    ('(252)(SO)🇸🇴 Somalia', '🇸🇴', '5911397852965244436'),
    ('(677)(SB)🇸🇧 Solomon Islands', '🇸🇧', '5911482712929080608'),
    ('(386)(SI)🇸🇮 Slovenia', '🇸🇮', '5913431983836368644'),
    ('(421)(SK)🇸🇰 Slovakia', '🇸🇰', '5913751666842145020'),
    ('(65)(SG)🇸🇬 Singapore', '🇸🇬', '5911531460808051849'),
    ('(232)(SL)🇸🇱 Sierra Leone', '🇸🇱', '5911210450657218661'),
    ('(248)(SC)🇸🇨 Seychelles', '🇸🇨', '5911185183364616913'),
    ('(381)(RS)🇷🇸 Serbia', '🇷🇸', '5913592598433369871'),
    ('(221)(SN)🇸🇳 Senegal', '🇸🇳', '5910995302860461643'),
    ('(239)(ST)🇸🇹 Sao Tome and Principe', '🇸🇹', '5913574331937462345'),
    ('(378)(SM)🇸🇲 San Marino', '🇸🇲', '5913587968458625465'),
    ('(685)(WS)🇼🇸 Samoa', '🇼🇸', '5913325971158602854'),
    ('(1)(KN)🇰🇳 Saint Kitts and Nevis', '🇰🇳', '5913691898077253637'),
    ('(1)(VC)🇻🇨 Saint Vincent and the Grenadines', '🇻🇨', '5911318941531116255'),
    ('(1)(LC)🇱🇨 Saint Lucia', '🇱🇨', '5911243659344351824'),
    ('(970)(PS)🇵🇸 Palestine', '🇵🇸', '5913684768431541668'),
    ('(250)(RW)🇷🇼 Rwanda', '🇷🇼', '5911455229433352234'),
    ('(40)(RO)🇷🇴 Romania', '🇷🇴', '5913460373570195273'),
    ('(974)(QA)🇶🇦 Qatar', '🇶🇦', '5911260864983339619'),
    ('(1)(PR)🇵🇷 Puerto Rico', '🇵🇷', '5911504350974317480'),
    ('(351)(PT)🇵🇹 Portugal', '🇵🇹', '5911023653939581472'),
    ('(63)(PH)🇵🇭 Philippines', '🇵🇭', '5911268638874145162'),
    ('(51)(PE)🇵🇪 Peru', '🇵🇪', '5911207993935925780'),
    ('(595)(PY)🇵🇾 Paraguay', '🇵🇾', '5911014265141072316'),
    ('(675)(PG)🇵🇬 Papua New Guinea', '🇵🇬', '5911107251183030903'),
    ('(507)(PA)🇵🇦 Panama', '🇵🇦', '5913428968769327174'),
    ('(680)(PW)🇵🇼 Palau', '🇵🇼', '5911283903187915549'),
    ('(92)(PK)🇵🇰 Pakistan', '🇵🇰', '5913705895375672082'),
    ('(968)(OM)🇴🇲 Oman', '🇴🇲', '5913570801474343473'),
    ('(47)(NO)🇳🇴 Norway', '🇳🇴', '5913617397574537046'),
    ('(234)(NG)🇳🇬 Nigeria', '🇳🇬', '5911143844304393105'),
    ('(227)(NE)🇳🇪 Niger', '🇳🇪', '5911270086278124251'),
    ('(64)(NZ)🇳🇿 New Zealand', '🇳🇿', '5913640044937089340'),
    ('(31)(NL)🇳🇱 Netherlands', '🇳🇱', '5913367645226275100'),
    ('(977)(NP)🇳🇵 Nepal', '🇳🇵', '5913496520014958723'),
    ('(264)(NA)🇳🇦 Namibia', '🇳🇦', '5911108535378252443'),
    ('(258)(MZ)🇲🇿 Mozambique', '🇲🇿', '5911333419865871464'),
    ('(212)(MA)🇲🇦 Morocco', '🇲🇦', '5911482111633658301'),
    ('(382)(ME)🇲🇪 Montenegro', '🇲🇪', '5913239436157522151'),
    ('(976)(MN)🇲🇳 Mongolia', '🇲🇳', '5911041383564580038'),
    ('(377)(MC)🇲🇨 Monaco', '🇲🇨', '5911245347266500057'),
    ('(373)(MD)🇲🇩 Moldova', '🇲🇩', '5913456847402045950'),
    ('(960)(MV)🇲🇻 Maldives', '🇲🇻', '5913501399097806832'),
    ('(223)(ML)🇲🇱 Mali', '🇲🇱', '5911305266355245916'),
    ('(356)(MT)🇲🇹 Malta', '🇲🇹', '5911023714069123567'),
    ('(52)(MX)🇲🇽 Mexico', '🇲🇽', '5913687302462246518'),
    ('(60)(MY)🇲🇾 Malaysia', '🇲🇾', '5913654360063087453'),
    ('(254)(KE)🇰🇪 Kenya', '🇰🇪', '5911154710571651231'),
    ('(261)(MG)🇲🇬 Madagascar', '🇲🇬', '5913766918271012920'),
    ('(389)(MK)🇲🇰 North Macedonia', '🇲🇰', '5913394029210374721'),
    ('(352)(LU)🇱🇺 Luxembourg', '🇱🇺', '5913390842344640293'),
    ('(370)(LT)🇱🇹 Lithuania', '🇱🇹', '5911172315642595523'),
    ('(423)(LI)🇱🇮 Liechtenstein', '🇱🇮', '5911166650580734660'),
    ('(218)(LY)🇱🇾 Libya', '🇱🇾', '5911236989260140996'),
    ('(231)(LR)🇱🇷 Liberia', '🇱🇷', '5913324167272337727'),
    ('(686)(KI)🇰🇮 Kiribati', '🇰🇮', '5911294443037660118'),
    ('(383)(XK)🇽🇰 Kosovo', '🇽🇰', '5911433681582429010'),
    ('(965)(KW)🇰🇼 Kuwait', '🇰🇼', '5913290705182134003'),
    ('(996)(KG)🇰🇬 Kyrgyzstan', '🇰🇬', '5911202161370337549'),
    ('(856)(LA)🇱🇦 Laos', '🇱🇦', '5913718526874489279'),
    ('(371)(LV)🇱🇻 Latvia', '🇱🇻', '5913738489882480243'),
    ('(961)(LB)🇱🇧 Lebanon', '🇱🇧', '5911504273664905447'),
    ('(266)(LS)🇱🇸 Lesotho', '🇱🇸', '5911059881988723711'),
    ('(62)(ID)🇮🇩 Indonesia', '🇮🇩', '5913479361620611038'),
    ('(98)(IR)🇮🇷 Iran', '🇮🇷', '5911308891307643032'),
    ('(964)(IQ)🇮🇶 Iraq', '🇮🇶', '5911382442622587735'),
    ('(353)(IE)🇮🇪 Ireland', '🇮🇪', '5913440715504881532'),
    ('(972)(IL)🇮🇱 Israel', '🇮🇱', '5911471936856134692'),
    ('(39)(IT)🇮🇹 Italy', '🇮🇹', '5913688444923547525'),
    ('(1)(JM)🇯🇲 Jamaica', '🇯🇲', '5913232280742006526'),
    ('(962)(JO)🇯🇴 Jordan', '🇯🇴', '5913234136167878475'),
    ('(354)(IS)🇮🇸 Iceland', '🇮🇸', '5911047899029967246'),
    ('(36)(HU)🇭🇺 Hungary', '🇭🇺', '5913767635530551104'),
    ('(504)(HN)🇭🇳 Honduras', '🇭🇳', '5911406889576436289'),
    ('(509)(HT)🇭🇹 Haiti', '🇭🇹', '5913459789454643194'),
    ('(592)(GY)🇬🇾 Guyana', '🇬🇾', '5913579412883771480'),
    ('(245)(GW)🇬🇼 Guinea-Bissau', '🇬🇼', '5911398694778836149'),
    ('(224)(GN)🇬🇳 Guinea', '🇬🇳', '5913471858312744319'),
    ('(502)(GT)🇬🇹 Guatemala', '🇬🇹', '5913324858762072330'),
    ('(1)(GD)🇬🇩 Grenada', '🇬🇩', '5913228063084121946'),
    ('(30)(GR)🇬🇷 Greece', '🇬🇷', '5911210399117611448'),
    ('(233)(GH)🇬🇭 Ghana', '🇬🇭', '5913391155877252952'),
    ('(995)(GE)🇬🇪 Georgia', '🇬🇪', '5913434771270144023'),
    ('(220)(GM)🇬🇲 Gambia', '🇬🇲', '5913657267755945883'),
    ('(241)(GA)🇬🇦 Gabon', '🇬🇦', '5911037896051137264'),
    ('(33)(FR)🇫🇷 France', '🇫🇷', '5913605586414473124'),
    ('(358)(FI)🇫🇮 Finland', '🇫🇮', '5911041344909873378'),
    ('(679)(FJ)🇫🇯 Fiji', '🇫🇯', '5911393832875856716'),
    ('(251)(ET)🇪🇹 Ethiopia', '🇪🇹', '5911078333168227043'),
    ('(1)(DO)🇩🇴 Dominican Republic', '🇩🇴', '5911152099231536123'),
    ('(670)(TL)🇹🇱 Timor-Leste', '🇹🇱', '5911141915864076479'),
    ('(593)(EC)🇪🇨 Ecuador', '🇪🇨', '5911273865849347408'),
    ('(20)(EG)🇪🇬 Egypt', '🇪🇬', '5913694831539916769'),
    ('(503)(SV)🇸🇻 El Salvador', '🇸🇻', '5913238624408703010'),
    ('(372)(EE)🇪🇪 Estonia', '🇪🇪', '5910986042910969906'),
    ('(1)(DM)🇩🇲 Dominica', '🇩🇲', '5911377121158107430'),
    ('(253)(DJ)🇩🇯 Djibouti', '🇩🇯', '5911407709915190157'),
    ('(45)(DK)🇩🇰 Denmark', '🇩🇰', '5911206009661034712'),
    ('(357)(CY)🇨🇾 Cyprus', '🇨🇾', '5911023550860366409'),
    ('(385)(HR)🇭🇷 Croatia', '🇭🇷', '5913692684056269311'),
    ('(506)(CR)🇨🇷 Costa Rica', '🇨🇷', '5911261745451635030'),
    ('(242)(CG)🇨🇬 Congo', '🇨🇬', '5911338788574990168'),
    ('(243)(CD)🇨🇩 DR Congo', '🇨🇩', '5913770362834783827'),
    ('(269)(KM)🇰🇲 Comoros', '🇰🇲', '5911338582416560604'),
    ('(855)(KH)🇰🇭 Cambodia', '🇰🇭', '5913699998385573485'),
    ('(237)(CM)🇨🇲 Cameroon', '🇨🇲', '5911172109484167745'),
    ('(1)(CA)🇨🇦 Canada', '🇨🇦', '5913623736946265914'),
    ('(238)(CV)🇨🇻 Cape Verde', '🇨🇻', '5913571501554012193'),
    ('(236)(CF)🇨🇫 Central African Republic', '🇨🇫', '5913443245240619222'),
    ('(235)(TD)🇹🇩 Chad', '🇹🇩', '5913299849167507310'),
    ('(420)(CZ)🇨🇿 Czechia', '🇨🇿', '5911198691036764307'),
    ('(56)(CL)🇨🇱 Chile', '🇨🇱', '5911471936856134692'),
    ('(57)(CO)🇨🇴 Colombia', '🇨🇴', '5913773060074246009'),
    ('(257)(BI)🇧🇮 Burundi', '🇧🇮', '5913766441529642752'),
    ('(267)(BW)🇧🇼 Botswana', '🇧🇼', '5911513782722499475'),
    ('(387)(BA)🇧🇦 Bosnia and Herzegovina', '🇧🇦', '5913700002680541032'),
    ('(591)(BO)🇧🇴 Bolivia', '🇧🇴', '5913638795101606133'),
    ('(975)(BT)🇧🇹 Bhutan', '🇧🇹', '5913236734623093021'),
    ('(229)(BJ)🇧🇯 Benin', '🇧🇯', '5913735869952430547'),
    ('(54)(AR)🇦🇷 Argentina', '🇦🇷', '5913573356979884082'),
    ('(61)(AU)🇦🇺 Australia', '🇦🇺', '5913632326880858455'),
    ('(43)(AT)🇦🇹 Austria', '🇦🇹', '5911338831524664592'),
    ('(1)(BS)🇧🇸 Bahamas', '🇧🇸', '5911451643135660214'),
    ('(973)(BH)🇧🇭 Bahrain', '🇧🇭', '5913581663446634403'),
    ('(880)(BD)🇧🇩 Bangladesh', '🇧🇩', '5911365056594973179'),
    ('(32)(BE)🇧🇪 Belgium', '🇧🇪', '5913529642802745141'),
    ('(501)(BZ)🇧🇿 Belize', '🇧🇿', '5913355005137522807'),
    ('(244)(AO)🇦🇴 Angola', '🇦🇴', '5913753316109586411'),
    ('(376)(AD)🇦🇩 Andorra', '🇦🇩', '5911314702398396902'),
    ('(213)(DZ)🇩🇿 Algeria', '🇩🇿', '5913782968563800236'),
    ('(355)(AL)🇦🇱 Albania', '🇦🇱', '5911357458797826163'),
    ('(93)(AF)🇦🇫 Afghanistan', '🇦🇫', '5913492040364068694'),
    ('(263)(ZW)🇿🇼 Zimbabwe', '🇿🇼', '5911092502265336396'),
    ('(53)(CU)🇨🇺 Cuba', '🇨🇺', '5431551436502611633'),
    ('(850)(KP)🇰🇵 North Korea', '🇰🇵', '5434142701941437163'),
    ('(58)(VE)🇻🇪 Venezuela', '🇻🇪', '5434009132753499322'),
    ('(963)(SY)🇸🇾 Syria', '🇸🇾', '5433910876786670092'),
    ('(95)(MM)🇲🇲 Myanmar', '🇲🇲', '5433666360003540231'),
    ('(505)(NI)🇳🇮 Nicaragua', '🇳🇮', '5334807849418003620'),
    ('(82)(KR)🇰🇷 South Korea', '🇰🇷', '5913371673905598425'),
    ('(240)(GQ)🇬🇶 Equatorial Guinea', '🇬🇶', '5911306279967529251'),
    ('(299)(GL)🇬🇱 Greenland', '🇬🇱', '5292014752283774878'),
    ('(298)(FO)🇫🇴 Faroe Islands', '🇫🇴', '5296469342039327674'),
    ("(225)(CI)🇨🇮 Côte d'Ivoire", '🇨🇮', '5222233374948602940'),
    ('(673)(BN)🇧🇳 Brunei', '🇧🇳', '5911336409163109113'),
    ('(359)(BG)🇧🇬 Bulgaria', '🇧🇬', '5294329219965272288'),
    ('(226)(BF)🇧🇫 Burkina Faso', '🇧🇫', '5913407764515786948'),
    ('(291)(ER)🇪🇷 Eritrea', '🇪🇷', '5433723401464198287'),
    ('(265)(MW)🇲🇼 Malawi', '🇲🇼', '5433968339154122439'),
    ('(222)(MR)🇲🇷 Mauritania', '🇲🇷', '5433859405898594234'),
    ('(674)(NR)🇳🇷 Nauru', '🇳🇷', '5434131139889478358'),
    ('(966)(SA)🇸🇦 Saudi Arabia', '🇸🇦', '4985897134424328239'),
    ('(676)(TO)🇹🇴 Tonga', '🇹🇴', '5433640100573491806'),
    ('(688)(TV)🇹🇻 Tuvalu', '🇹🇻', '5433684690923961019'),
    ('(886)(TW)🇹🇼 Taiwan', '🇹🇼', '5366187256937726720'),
    ('(852)(HK)🇭🇰 Hong Kong', '🇭🇰', '5292166459118606932'),
    ('(853)(MO)🇲🇴 Macau', '🇲🇴', '6323557758096377611'),
    ('(262)(YT)🇾🇹 Mayotte', '🇾🇹', '5780471598922337683'),
    ('(687)(NC)🇳🇨 New Caledonia', '🇳🇨', '5780471598922337683'),
    ('(683)(NU)🇳🇺 Niue', '🇳🇺', '5780471598922337683'),
    ('(672)(NF)🇳🇫 Norfolk Island', '🇳🇫', '5780471598922337683'),
    ('(64)(PN)🇵🇳 Pitcairn Islands', '🇵🇳', '5780471598922337683'),
    ('(262)(RE)🇷🇪 Reunion', '🇷🇪', '5780471598922337683'),
    ('(290)(SH)🇸🇭 Saint Helena', '🇸🇭', '5780471598922337683'),
    ('(690)(TK)🇹🇰 Tokelau', '🇹🇰', '5780471598922337683'),
    ('(1)(TC)🇹🇨 Turks and Caicos Islands', '🇹🇨', '5780471598922337683'),
    ('(1)(VI)🇻🇮 US Virgin Islands', '🇻🇮', '5780471598922337683'),
    ('(681)(WF)🇼🇫 Wallis and Futuna', '🇼🇫', '5780471598922337683'),
    ('(212)(EH)🇪🇭 Western Sahara', '🇪🇭', '5780471598922337683'),
    ('(682)(CK)🇨🇰 Cook Islands', '🇨🇰', '5780471598922337683'),
    ('(689)(PF)🇵🇫 French Polynesia', '🇵🇫', '5780471598922337683'),
    ('(350)(GI)🇬🇮 Gibraltar', '🇬🇮', '5780471598922337683'),
    ('(47)(SJ)🇸🇯 Svalbard and Jan Mayen', '🇸🇯', '5780471598922337683'),
    ('(358)(AX)🇦🇽 Aland Islands', '🇦🇽', '5780471598922337683'),
    ('(44)(JE)🇯🇪 Jersey', '🇯🇪', '5780471598922337683'),
    ('(44)(GG)🇬🇬 Guernsey', '🇬🇬', '5780471598922337683'),
    ('(44)(IM)🇮🇲 Isle of Man', '🇮🇲', '5226538255029121667'),
    ('(508)(PM)🇵🇲 Saint Pierre and Miquelon', '🇵🇲', '5780471598922337683'),
    ('(1)(SX)🇸🇽 Sint Maarten', '🇸🇽', '5461113820955027461'),
    ('(599)(BQ)🇧🇶 Bonaire', '🇧🇶', '5780471598922337683'),
]

SERVICES = [('📘 Facebook', '5334807341109908955'), ('💬 WhatsApp', '5334759662677957452'), ('📸 Instagram', '5334868205091459431'), ('✈️ Telegram', '5337010556253543833'), ('💄 HSBC', '5161519678597629383'), ('💭 Imo', '5337155807752524558'), ('🍎 Apple', '5334637951894722661'), ('🔍 Google', '5335010201005231986'), ('🪟 Microsoft', '5334880948259427772'), ('🧑\u200d🤝\u200d🧑 Teams', '5334590977837403844'), ('🎵 Tiktok', '5339213256001102461'), ('🏦 Bkash', '5348469219761626211'), ('🚀 Rocket', '5346042941196507141'), ('📈 Bybit', '5348372939479751825'), ('💱 Binance', '5348212415077064131'), ('🌟 Melbet', '5337102391244263212'), ('👻 Snapchat', '5359441366554255082'), ('🚗 Uber', '5298715455316303708'), ('💵 PayPal', '5776103539872896061'), ('🎬 Discord', '5116246243646898866'), ('🌟 Amazon', '4995019580536524226'), ('💜 Viber', '5463060437572528782'), ('💼 Linkedin', '6224222994265279792'), ('🔒 Line', '5399818044866327279'), ('🌟 Wechat', '5782757599560602950'), ('🐦 Twitter', '5215726959056662534'), ('👽 Reddit', '4992421103847604984'), ('📌 Pinterest', '5346103513120258857'), ('🎮 Twitch', '5233333563306301418'), ('📹 Zoom', '5881799193219043268'), ('💬 Signal', '5293998404404272267'), ('💻 Slack', '4994972469040251302'), ('☎️ Skype', '4992613535562334989'), ('🎥 Netflix', '6255738712664050133'), ('🎵 Spotify', '5411392711146095115'), ('📺 Amazon Prime', '6111801057061374810'), ('🍿 Hoichoi', '6104822598493801746'), ('📦 Daraz', '5336879280578138635'), ('🐼 Foodpanda', '5336879280578138635'), ('🛵 Pathao', '5336879280578138635'), ('🛒 AliExpress', '5336879280578138635'), ('🛍️ Shopee', '5336879280578138635'), ('💳 Payoneer', '5336879280578138635'), ('🦉 Wise', '5336879280578138635'), ('🤖 ChatGPT', '5296516998996445955'), ('📓 Notion', '5336879280578138635'), ('🐙 GitHub', '5417836094098007862'), ('🖌️ Canva', '5111661409008092227'), ('🎨 Figma', '5336879280578138635'), ('💼 Upwork', '5336879280578138635'), ('🟢 Fiverr', '5336879280578138635'), ('🌐 Yahoo', '5336879280578138635'), ('☁️ Dropbox', '5336879280578138635'), ('📚 Coursera', '5336879280578138635'), ('🗣️ Duolingo', '5336879280578138635'), ('📱 QSMS', '6321257309188136835'), ('📸 INSTAGRAM', '5334868205091459431'), ('💄 HSBC', '5161519678597629383'), ('🎵 TIK TOK', '5327982530702359565'), ('🩵 TIK TOK', '5391044040860906456')]

# Per-user setup state and per-country running configuration.
users = {}
# key -> {label, flag, emoji_id, interval, length, running}
country_cfg = {}
AUTO_TASKS = {}
DATA_FILE = "countries_demo.json"
METHOD_URL = "https://t.me/range_channele"
NUMBER_URL = "https://t.me/TSNumber9Bot?start=6136815573"
LANGUAGES = {"HI": "Hindi", "AR": "Arabic", "EN": "English", "ES": "Spanish"}
PAGE_SIZE = 12


def custom_emoji(emoji, emoji_id):
    return f'<tg-emoji emoji-id="{emoji_id}">{emoji}</tg-emoji>'


def otp(length):
    # Local demo generator only; not a real verification code.
    return "".join(random.choice("0123456789") for _ in range(length))


def country_short(country):
    label = country[0]
    m = re.search(r'\(([A-Za-z]{2})\)', label)
    return m.group(1).upper() if m else "XX"


def country_code(country):
    label = country[0]
    m = re.search(r'\((\d+)\)', label)
    return m.group(1) if m else "000"


def country_display(country):
    label, flag, eid = country
    return f"{custom_emoji(flag, eid)} <b>#{country_short(country)}</b>"


def save_countries():
    """Persist only countries explicitly added by the admin."""
    data = {}
    for key, item in country_cfg.items():
        data[key] = {
            "country": list(item["country"]),
            "interval": int(item.get("interval", 10)),
            "length": int(item.get("length", 6)),
            "language": item.get("language", "EN"),
            "running": bool(item.get("running", False)),
        }
    try:
        with open(DATA_FILE, "w", encoding="utf-8") as f:
            json.dump(data, f, ensure_ascii=False, indent=2)
    except Exception as e:
        logging.warning("Could not save countries: %s", e)


def load_countries():
    """Load previously added countries; do not preload the full country list."""
    try:
        with open(DATA_FILE, "r", encoding="utf-8") as f:
            data = json.load(f)
        for key, item in data.items():
            country = tuple(item["country"])
            if len(country) == 3:
                country_cfg[key] = {
                    "country": country,
                    "interval": int(item.get("interval", 10)),
                    "length": int(item.get("length", 6)),
                    "language": item.get("language", "EN"),
                    "running": False,  # never auto-start after restart
                }
    except FileNotFoundError:
        pass
    except Exception as e:
        logging.warning("Could not load countries: %s", e)


load_countries()


def country_keyboard(page=0):
    # IMPORTANT: show ONLY countries added by the admin.
    items = list(country_cfg.values())
    start = page * PAGE_SIZE
    rows = items[start:start + PAGE_SIZE]
    buttons = []
    for item in rows:
        c = item["country"]
        key = country_short(c)
        state = "ON" if item["running"] else "OFF"
        buttons.append([
            InlineKeyboardButton(
                f"#{key} • {state} • {item['interval']}s / {item['length']}d",
                callback_data=f"country:{key}"
            ),
            InlineKeyboardButton("🗑️", callback_data=f"delcountry:{key}")
        ])
    nav = []
    if page > 0:
        nav.append(InlineKeyboardButton("PREVIOUS", callback_data=f"cpage:{page-1}"))
    if (page + 1) * PAGE_SIZE < len(items):
        nav.append(InlineKeyboardButton("NEXT", callback_data=f"cpage:{page+1}"))
    if nav:
        buttons.append(nav)
    if len(items) == 0:
        buttons.append([InlineKeyboardButton("NO COUNTRY ADDED", callback_data="noop")])
    if ADMIN_ID:
        buttons.append([InlineKeyboardButton("ADD COUNTRY", callback_data="admin:add")])
    return InlineKeyboardMarkup(buttons)


def service_keyboard(page=0):
    start = page * PAGE_SIZE
    rows = SERVICES[start:start + PAGE_SIZE]
    buttons = []
    for idx, (label, eid) in enumerate(rows, start=start):
        emoji = label.split()[0]
        name  = " ".join(label.split()[1:])
        btn_text = f'{label}'
        buttons.append([InlineKeyboardButton(btn_text, callback_data=f"service:{idx}")])
    nav = []
    if page > 0:
        nav.append(InlineKeyboardButton("◀️ PREV", callback_data=f"spage:{page-1}"))
    if (page + 1) * PAGE_SIZE < len(SERVICES):
        nav.append(InlineKeyboardButton("NEXT ▶️", callback_data=f"spage:{page+1}"))
    if nav:
        buttons.append(nav)
    return InlineKeyboardMarkup(buttons)


def admin_keyboard():
    return InlineKeyboardMarkup([
        [InlineKeyboardButton("ADD COUNTRY", callback_data="admin:add")],
        [InlineKeyboardButton("DELETE COUNTRY", callback_data="admin:delete")],
        [InlineKeyboardButton("START ALL", callback_data="admin:startall"),
         InlineKeyboardButton("STOP ALL", callback_data="admin:stopall")],
    ])


def interval_keyboard(key):
    return InlineKeyboardMarkup([
        [InlineKeyboardButton("5 SEC", callback_data=f"interval:{key}:5"),
         InlineKeyboardButton("10 SEC", callback_data=f"interval:{key}:10")],
        [InlineKeyboardButton("15 SEC", callback_data=f"interval:{key}:15"),
         InlineKeyboardButton("20 SEC", callback_data=f"interval:{key}:20")],
    ])


def length_keyboard(key):
    return InlineKeyboardMarkup([
        [InlineKeyboardButton("4 DIGIT", callback_data=f"length:{key}:4"),
         InlineKeyboardButton("5 DIGIT", callback_data=f"length:{key}:5")],
        [InlineKeyboardButton("6 DIGIT", callback_data=f"length:{key}:6"),
         InlineKeyboardButton("8 DIGIT", callback_data=f"length:{key}:8")],
    ])


def running_keyboard():
    buttons = []
    for key, item in country_cfg.items():
        state = "ON" if item["running"] else "OFF"
        buttons.append([
            InlineKeyboardButton(
                f"#{key} • {state} • {item['interval']}s / {item['length']}d",
                callback_data=f"toggle:{key}"
            ),
            InlineKeyboardButton(
                f"REMOVE #{key}", callback_data=f"remove:{key}"
            ),
        ])
    buttons.append([InlineKeyboardButton("ADD COUNTRY", callback_data="admin:add")])
    return InlineKeyboardMarkup(buttons)


def language_keyboard(key):
    return InlineKeyboardMarkup([
        [InlineKeyboardButton("HI", callback_data=f"lang:{key}:HI"),
         InlineKeyboardButton("AR", callback_data=f"lang:{key}:AR")],
        [InlineKeyboardButton("EN", callback_data=f"lang:{key}:EN"),
         InlineKeyboardButton("ES", callback_data=f"lang:{key}:ES")],
    ])


def make_test_number(country):
    # Display-only demo number; no real number is acquired.
    code = country_code(country)
    suffix = "".join(random.choice("0123456789") for _ in range(4))
    return f"+{code}{suffix}"


def service_short(service_label):
    """Create a compact 3-letter service tag for the display line."""
    name = service_label.split(maxsplit=1)[-1].strip()
    letters = re.sub(r"[^A-Za-z]", "", name).upper()
    return letters[:3] if letters else "MSI"


def build_demo_post(item, service):
    country = item["country"]
    service_label, service_id = service
    code = otp(item["length"])
    number = make_test_number(country)
    flag, flag_id = country[1], country[2]

    cc = country_code(country)
    suffix = number[len(cc) + 1:] if number.startswith("+") else number
    tag = service_short(service_label)

    # Keep custom-emoji tags OUTSIDE <pre>/<code>.
    # Telegram can otherwise render the fallback Unicode emoji instead of
    # the supplied premium/custom emoji.
    # Clean Telegram-friendly box: real line breaks, no literal \\n text.
    # Custom emojis are kept outside <pre>/<code> so Telegram can render them.
    green_emoji = '<tg-emoji emoji-id="5210931095494733350">🟢</tg-emoji>'
    msg = (
        f"{custom_emoji(service_label.split()[0], service_id)} "
        f"{custom_emoji(flag, flag_id)} #{country_short(country)} "
        f"{cc} {green_emoji} {suffix} #EN"
    )

    keyboard = InlineKeyboardMarkup([
        [
            InlineKeyboardButton(
                f"{code}",
                copy_text=CopyTextButton(code),
                style="success",
            ),
            InlineKeyboardButton(
                "METHOD",
                url=METHOD_URL,
                style="success",
            ),
        ],
        [
            InlineKeyboardButton(
                "GET NUMBER",
                url=NUMBER_URL,
                style="primary",
            )
        ],
    ])
    return msg, keyboard


async def post_one(app, key):
    item = country_cfg.get(key)
    if not item or not item["running"] or not GROUP_ID:
        return
    # Country-র নিজস্ব service ব্যবহার করবে
    svc_label = item.get("service_label")
    svc_id    = item.get("service_id")
    if svc_label and svc_id:
        service = (svc_label, svc_id)
    else:
        service = SERVICES[0]  # default Facebook
    msg, keyboard = build_demo_post(item, service)
    try:
        await app.bot.send_message(
            chat_id=GROUP_ID,
            text=msg,
            parse_mode=ParseMode.HTML,
            reply_markup=keyboard,
        )
    except Exception as e:
        logging.warning("Group send failed for %s: %s", key, e)


async def country_loop(app, key):
    while country_cfg.get(key, {}).get("running"):
        await post_one(app, key)
        await asyncio.sleep(country_cfg[key]["interval"])
    AUTO_TASKS.pop(key, None)


async def ensure_country_task(app, key):
    if key not in AUTO_TASKS or AUTO_TASKS[key].done():
        AUTO_TASKS[key] = app.create_task(country_loop(app, key))


async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    # Keep the button text exactly equal to the text_handler checks below.
    kb = ReplyKeyboardMarkup(
        [["COUNTRY SELECT", "DELETE COUNTRY"],
         ["GENERATE TEST OTP"]],
        resize_keyboard=True,
    )
    await update.message.reply_text(
        "<b>Welcome to OTP Send Panel</b>\n\n"
        "<b>DEMO / TEST MODE</b>\n"
        "Codes are generated locally; no real OTP or phone number is received.",
        parse_mode=ParseMode.HTML,
        reply_markup=kb,
    )


async def country_select(update: Update, context: ContextTypes.DEFAULT_TYPE):
    count = len(country_cfg)
    title = (
        f"<b>ADDED COUNTRIES</b> — {count}" if count
        else "<b>ADDED COUNTRIES</b> — 0\n\nNo country added yet. Admin can add one below."
    )
    await update.message.reply_text(
        title,
        parse_mode=ParseMode.HTML,
        reply_markup=country_keyboard(0),
    )


async def delete_country(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if update.effective_user.id != ADMIN_ID:
        await update.message.reply_text("⛔ Admin only.")
        return
    running = sum(1 for x in country_cfg.values() if x["running"])
    await update.message.reply_text(
        f"<b>COUNTRY CONTROL</b>\n\nRunning: <b>{running}</b>\n"
        "Tap a country button to turn demo OTP posting ON/OFF.",
        parse_mode=ParseMode.HTML,
        reply_markup=running_keyboard(),
    )


async def callbacks(update: Update, context: ContextTypes.DEFAULT_TYPE):
    q = update.callback_query
    await q.answer()
    uid = q.from_user.id
    data = q.data

    if data == "noop":
        return

    if data.startswith("cpage:"):
        await q.edit_message_reply_markup(reply_markup=country_keyboard(int(data.split(":")[1])))
        return
    if data.startswith("spage:"):
        await q.edit_message_reply_markup(reply_markup=service_keyboard(int(data.split(":")[1])))
        return

    if data.startswith("country:"):
        key = data.split(":", 1)[1]
        if key not in country_cfg:
            await q.edit_message_text("❌ Country not found.")
            return
        users.setdefault(uid, {})["country_key"] = key
        c = country_cfg[key]["country"]
        await q.edit_message_text(
            f"✅ <b>Country selected</b>\n\n{country_display(c)}\n\n"
            "📱 <b>Select Service:</b>",
            parse_mode=ParseMode.HTML,
            reply_markup=service_keyboard(),
        )
        return

    if data.startswith("delcountry:"):
        if uid != ADMIN_ID:
            await q.answer("⛔ Admin only", show_alert=True)
            return
        key = data.split(":", 1)[1]
        if key not in country_cfg:
            await q.answer("❌ Not found", show_alert=True)
            return
        country_cfg[key]["running"] = False
        del country_cfg[key]
        save_countries()
        await q.answer("✅ Deleted!", show_alert=True)
        await q.edit_message_reply_markup(reply_markup=country_keyboard(0))
        return

    if data.startswith("lang:"):
        _, key, lang = data.split(":")
        if uid != ADMIN_ID:
            await q.answer("Admin only", show_alert=True)
            return
        if key not in country_cfg or lang not in LANGUAGES:
            await q.answer("Invalid selection", show_alert=True)
            return
        country_cfg[key]["language"] = lang
        save_countries()
        users.setdefault(uid, {})["country_key"] = key
        await q.edit_message_text(
            f"✅ <b>ভাষা নির্বাচন:</b> {lang}\n\n"
            "🔢 <b>কত ডিজিটের OTP পাঠাবেন?</b>",
            parse_mode=ParseMode.HTML,
            reply_markup=length_keyboard(key),
        )
        return

    if data.startswith("interval:"):
        _, key, sec = data.split(":")
        if uid != ADMIN_ID:
            await q.answer("Admin only", show_alert=True)
            return
        country_cfg[key]["interval"] = int(sec)
        country_cfg[key]["running"] = True
        save_countries()
        users.setdefault(uid, {})["country_key"] = key
        await ensure_country_task(context.application, key)
        svc = country_cfg[key].get("service_label", "—")
        await q.edit_message_text(
            f"✅ <b>সব সেটআপ সম্পন্ন!</b>\n\n"
            f"🌍 Country: <b>#{key}</b>\n"
            f"📱 Service: <b>{svc}</b>\n"
            f"🔢 Digits: <b>{country_cfg[key]['length']}</b>\n"
            f"⏱ Interval: <b>{sec} seconds</b>\n\n"
            "🟢 OTP পাঠানো শুরু হয়েছে!",
            parse_mode=ParseMode.HTML,
        )
        return

    if data.startswith("length:"):
        _, key, length = data.split(":")
        if uid != ADMIN_ID:
            await q.answer("Admin only", show_alert=True)
            return
        country_cfg[key]["length"] = int(length)
        save_countries()
        await q.edit_message_text(
            f"✅ <b>OTP Digits:</b> {length}\n\n"
            "⏱ <b>কত সেকেন্ড পর পর OTP পাঠাবেন?</b>",
            parse_mode=ParseMode.HTML,
            reply_markup=interval_keyboard(key),
        )
        return

    if data == "admin:add":
        if uid != ADMIN_ID:
            await q.answer("Admin only", show_alert=True)
            return
        users.setdefault(uid, {})["awaiting_add"] = True
        await q.edit_message_text(
            "➕ <b>ADD COUNTRY</b>\n\n"
            "শুধু Country ISO code পাঠাও, যেমন: <code>MG</code>\n\n"
            "<b>MG</b> দিলে অটোমেটিকভাবে Madagascar select হবে।\n"
            "একইভাবে US, BD, ZW, KE, EG — যেকোনো তালিকাভুক্ত Country code দিতে পারবে।\n\n"
            "তারপর interval এবং OTP digit length select করবে।\n"
            "This is display-only demo mode; no real phone number is used.",
            parse_mode=ParseMode.HTML,
        )
        return

    if data == "admin:delete":
        if uid != ADMIN_ID:
            await q.answer("Admin only", show_alert=True)
            return
        await q.edit_message_text("🗑️ <b>COUNTRY CONTROL</b>", parse_mode=ParseMode.HTML, reply_markup=running_keyboard())
        return

    if data in ("admin:startall", "admin:stopall"):
        if uid != ADMIN_ID:
            await q.answer("Admin only", show_alert=True)
            return
        on = data.endswith("startall")
        for key, item in country_cfg.items():
            item["running"] = on
            if on:
                await ensure_country_task(context.application, key)
        save_countries()
        await q.edit_message_text(
            f"{'🟢 All demo countries started.' if on else '⏹ All demo countries stopped.'}",
            reply_markup=running_keyboard(),
        )
        return

    if data.startswith("remove:"):
        if uid != ADMIN_ID:
            await q.answer("Admin only", show_alert=True)
            return
        key = data.split(":", 1)[1]
        item = country_cfg.pop(key, None)
        task = AUTO_TASKS.pop(key, None)
        if task and not task.done():
            task.cancel()
        save_countries()
        if users.get(uid, {}).get("country_key") == key:
            users[uid].pop("country_key", None)
        await q.edit_message_text(
            f"Country <b>#{key}</b> removed successfully.",
            parse_mode=ParseMode.HTML,
            reply_markup=running_keyboard(),
        )
        return

    if data.startswith("toggle:"):
        if uid != ADMIN_ID:
            await q.answer("Admin only", show_alert=True)
            return
        key = data.split(":", 1)[1]
        item = country_cfg.get(key)
        if not item:
            return
        item["running"] = not item["running"]
        save_countries()
        if item["running"]:
            await ensure_country_task(context.application, key)
        await q.edit_message_reply_markup(reply_markup=running_keyboard())
        return

    if data.startswith("service:"):
        idx = int(data.split(":")[1])
        label, eid = SERVICES[idx]
        users.setdefault(uid, {})["service"] = (label, eid)
        key = users[uid].get("country_key")
        if not key:
            await q.edit_message_text("❌ আগে Country add করুন।")
            return
        country_cfg[key]["service_label"] = label
        country_cfg[key]["service_id"] = eid
        save_countries()
        await q.edit_message_text(
            f"✅ <b>Service selected:</b> {label}\n\n"
            "🌐 <b>ভাষা নির্বাচন করুন:</b>",
            parse_mode=ParseMode.HTML,
            reply_markup=language_keyboard(key),
        )
        return


async def generate(update: Update, context: ContextTypes.DEFAULT_TYPE):
    uid = update.effective_user.id
    data = users.get(uid, {})
    key = data.get("country_key")
    if not key or key not in country_cfg:
        await update.message.reply_text("আগে Country Select করুন।")
        return
    item = country_cfg[key]
    service = data.get("service") or SERVICES[0]
    msg, keyboard = build_demo_post(item, service)
    await update.message.reply_text(msg, parse_mode=ParseMode.HTML, reply_markup=keyboard)


async def admin(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if update.effective_user.id != ADMIN_ID:
        await update.message.reply_text("⛔ Admin only.")
        return
    running = sum(1 for x in country_cfg.values() if x["running"])
    await update.message.reply_text(
        f"⚙️ <b>ADMIN PANEL</b>\n\n🌍 Countries: <b>{len(country_cfg)}</b>\n"
        f"🟢 Running: <b>{running}</b>\n\n"
        "Choose an action below.",
        parse_mode=ParseMode.HTML,
        reply_markup=admin_keyboard(),
    )


async def text_handler(update: Update, context: ContextTypes.DEFAULT_TYPE):
    uid = update.effective_user.id
    text = update.message.text.strip()

    if text == "COUNTRY SELECT":
        await country_select(update, context)
        return
    if text == "DELETE COUNTRY":
        await delete_country(update, context)
        return
    if text == "SERVICE SELECT":
        await update.message.reply_text(
            "<b>SELECT SERVICE</b>",
            parse_mode=ParseMode.HTML,
            reply_markup=service_keyboard(0),
        )
        return
    if text == "GENERATE TEST OTP":
        await generate(update, context)
        return

    if uid == ADMIN_ID and users.get(uid, {}).get("awaiting_add"):
        # Simple country add: the admin only sends the ISO code, e.g. MG.
        # The full country name, calling code, flag and custom emoji ID are
        # automatically taken from the built-in COUNTRIES list.
        raw = text.strip().upper()
        iso = None

        # Accept: MG, #MG, 261|MG, 261 MG, or 261|MG|Madagascar.
        m = re.search(r"\b([A-Z]{2})\b", raw)
        if m:
            iso = m.group(1)

        country_match = None
        if iso:
            for c in COUNTRIES:
                if country_short(c) == iso:
                    country_match = c
                    break

        if country_match is None:
            await update.message.reply_text(
                "❌ Country code পাওয়া যায়নি।\n\n"
                "শুধু ISO code পাঠাও, যেমন: <code>MG</code>\n"
                "MG = Madagascar",
                parse_mode=ParseMode.HTML,
            )
            return

        country_cfg[iso] = {
            "country": country_match,
            "interval": 10,
            "length": 6,
            "language": "EN",
            "running": False,
        }
        save_countries()
        users[uid]["awaiting_add"] = False
        users[uid]["country_key"] = iso
        await update.message.reply_text(
            f"✅ Added <b>#{iso}</b> — {country_match[0]}\n\n"
            "📱 <b>কোন Service-এর OTP পাঠাবেন?</b>",
            parse_mode=ParseMode.HTML,
            reply_markup=service_keyboard(),
        )
        return

    await update.message.reply_text("Use /start to open the panel.")


async def run_bot():
    app = Application.builder().token(BOT_TOKEN).build()
    app.add_handler(CommandHandler("start", start))
    app.add_handler(CommandHandler("generate", generate))
    app.add_handler(CommandHandler("admin", admin))
    app.add_handler(CallbackQueryHandler(callbacks))
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, text_handler))
    logging.info("🤖 Bot polling started...")
    async with app:
        await app.start()
        await app.updater.start_polling()
        await asyncio.Event().wait()


def main():
    # Flask keep-alive আলাদা daemon thread-এ চলবে
    t = threading.Thread(target=run_flask, daemon=True)
    t.start()
    logging.info("🌐 Flask server started on port %s", os.environ.get('PORT', 8080))

    # Bot main thread-এ asyncio.run() দিয়ে চলবে
    asyncio.run(run_bot())


if __name__ == "__main__":
    main()
