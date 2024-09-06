# Agent Internals 

## Introduction 

The main tasks of the Paraxial.io agent are:

1. Record incoming HTTP requests and send them to the Paraxial.io backend for processing

2. Enforce bans against IP addresses 

3. Enforce rules with a time period of < 30 seconds, using local ETS tables for state

## Information Sent/Received by Agent

Sent to backend:
- Bundles of HTTP requests, Paraxial.HTTPBuffer.send_http(state), 

Retrieved from backend for local use:
- Allow list
- Ban list
- Local rules

Important values:
- How often are HTTP request bundles sent to backend? (3 seconds)
- How often is the allow/ban/rules request sent? (3 seconds)
- Max length of local rules (30 seconds)
- Local bans cleared (30 seconds)


Paraxial.io backend:
@local_rule_max_seconds 30
- only rules with a time period < @local_rule_max_seconds are sent to agent

## Plug -> Module -> Function Map

`AllowedPlug -> Paraxial.Crow.eval_http(conn)`

`RecordPlug -> Paraxial.HTTPBuffer.add_http_event(conn)`

## ETS Tables

Crow:
- `:backend_bans`
- `:local_bans`
- `:rule_names`

LocalRule:
- `:local_rule_n`
- ets_atom is used for table name, created as `:"local_rule_#{rule.id}"`

