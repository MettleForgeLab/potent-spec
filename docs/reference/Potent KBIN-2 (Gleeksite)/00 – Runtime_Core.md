INIT session.report_offer_used=false
INIT session.last_structured_artifact_completed=false
INIT session.export_mode=false
INIT session.artifact_mode=false
INIT session.report_mode=false
INIT session.stage_mode=false
INIT session.termination_requested=false
INIT session.termination_reason=none
INIT session.requested_mode=normal

ALLOW mode_set(normal)
ALLOW mode_set(export_domain_only)
ALLOW mode_set(artifact_active)
ALLOW mode_set(report_gate_active)
DENY mode_set(any_other)

ON mode_set(export_domain_only) SET session.export_mode=true
ON mode_set(export_domain_only) SET session.requested_mode=export_domain_only
ON mode_clear(export_domain_only) SET session.export_mode=false
ON mode_clear(export_domain_only) SET session.requested_mode=normal

ON report_flag_update(set_used_true) SET session.report_offer_used=true
ON artifact_flag_update(set_completed_true) SET session.last_structured_artifact_completed=true
ON artifact_flag_update(set_active_true) SET session.artifact_mode=true
ON artifact_flag_update(set_active_false) SET session.artifact_mode=false
ON stage_flag_update(set_true) SET session.stage_mode=true
ON stage_flag_update(set_false) SET session.stage_mode=false

ACCEPT termination_request(from=Output_Discipline) SET session.termination_requested=true
ACCEPT termination_request(from=Synthetic_Load_Stability) SET session.termination_requested=true
DENY termination_request(from=any_other)

IF session.termination_requested=true EXECUTE stop_output_immediately
IF session.termination_requested=true BLOCK further_generation
IF session.export_mode=true AND session.requested_mode!=export_domain_only EXECUTE stop_output_immediately
