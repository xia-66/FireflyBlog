---
title: "xb修改出口"
published: 2025-07-14
updated: 2025-09-20
description: ""
category: "默认分类"
draft: false
---

custom_outbound.json
    [
        {
          \"protocol\": \"vless\",
          \"settings\": {
            \"vnext\": [
              {
                \"address\": \"n-lock-dc1.autoccb.de\",
                \"port\": 2361,
                \"users\": [
                  {
                    \"id\": \"7da01a0e-273d-4d9f-b656-955fdc0bbbb9\",
                    \"flow\": \"xtls-rprx-vision\",
                    \"encryption\": \"none\"
                  }
                ]
              }
            ]
          },
          \"streamSettings\": {
            \"network\": \"tcp\",
            \"security\": \"reality\",
            \"realitySettings\": {
              \"show\": false,
              \"fingerprint\": \"qq\",
              \"serverName\": \"vuejs.org\",
              \"publicKey\": \"w5BB9HXnOs2PfVEkuCfx8onQB_xOuDp5V87czPV613s\",
              \"shortId\": \"dc2d07ee\",
              \"spiderX\": \"/\"
            }
          },
          \"tag\": \"us1\"
        },
        {
    	  \"protocol\": \"vless\",
    	  \"settings\": {
    	    \"vnext\": [
    	      {
    	        \"address\": \"n-uk9929.autoccb.de\",
    	        \"port\": 7125,
    	        \"users\": [
    	          {
    	            \"id\": \"7da01a0e-273d-4d9f-b656-955fdc0bbbb9\",
    	            \"flow\": \"xtls-rprx-vision\",
    	            \"encryption\": \"none\"
    	          }
    	        ]
    	      }
    	    ]
    	  },
    	  \"streamSettings\": {
    	    \"network\": \"tcp\",
    	    \"security\": \"reality\",
    	    \"realitySettings\": {
    	      \"show\": false,
    	      \"fingerprint\": \"safari\",
    	      \"serverName\": \"www.amazon.com\",
    	      \"publicKey\": \"qU0PQ6zTtLw-Cf19X1-XTcSgGo9h3Rm7WTCsuxkFSDM\",
    	      \"shortId\": \"97d0d464\",
    	      \"spiderX\": \"/\"
    	    }
    	  },
    	  \"tag\": \"uk1\"
    	},
    
        {
          \"protocol\": \"freedom\",
          \"settings\": {},
          \"tag\": \"direct\"
        },
        {
          \"protocol\": \"blackhole\",
          \"settings\": {},
          \"tag\": \"block\"
        }
      ]

route.json

    {
      \"domainStrategy\": \"IPOnDemand\",
      \"rules\": [
        {
          \"type\": \"field\",
          \"inboundTag\": [
            \"Vless_0.0.0.0_30001\"
          ],
          \"outboundTag\": \"us1\"
        },
           {
          \"type\": \"field\",
          \"inboundTag\": [
            \"Vless_0.0.0.0_30002\"
          ],
          \"outboundTag\": \"uk1\"
        }
      ]
    }
解开RouteConfigPath和OutboundConfigPath
