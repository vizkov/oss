# Framework Coverage & Enforcement

## Layer Coverage

- [ ] **Data Layer**: Both SOURCE and SINK tags present for each data flow?
- [ ] **Logic Layer**: Both LOGIC rules and FLOW steps documented?
- [ ] **Analysis Layer**: Both AREA surface and BUG findings present?
- [ ] **Organization Layer**: AUDIT context at file level? REVIEW notes for unclear items?
- [ ] **Structural Layer**: SECTION boundaries used? LINK connections present?
- [ ] **Metadata Layer**: RELATION patterns documented? INSTANCE occurrences marked?

## Relationship Pattern Coverage

- [ ] **Data Flow**: Every SOURCE has corresponding SINK?
- [ ] **Mirroring**: Symmetric operations paired (sign ↔ verify)?
- [ ] **Lifecycle**: Sequential steps properly ordered (seq)?
- [ ] **Parent**: Sections properly contain related tags?
- [ ] **Reference**: Bugs linked to relevant areas?
- [ ] **Validation**: Logic rules linked to enforcement code?
- [ ] **Exploit Chain**: Bugs link to all contributing factors?
- [ ] **Remediation**: Bugs have fix suggestions in REVIEW?
- [ ] **Assumption**: Cross-file dependencies documented?
- [ ] **AltPath**: Branches (if/else) both documented?
- [ ] **Composition**: Complex instances broken into phases?
- [ ] **Intersection**: Combined patterns documented?

## Metadata Layer Detailed Checks

- [ ] **SEC-RELATION presence**: At least one pattern type documented?
- [ ] **SEC-INSTANCE presence**: At least one pattern instance marked?
- [ ] **RELATION → INSTANCE pairing**: Every RELATION has corresponding INSTANCE(s)?
- [ ] **Epic consistency**: RELATION and INSTANCE use same epic values?
- [ ] **Composition structure**: Parent INSTANCE has child instances linked?
- [ ] **Intersection structure**: Combined pattern links to source patterns?
- [ ] **Pattern documentation**: Each relationship type (Data_Flow, Mirroring, etc.) has RELATION if used?

## Syntax Compliance

- [ ] All anchors have `epic` attribute?
- [ ] All anchors have emojis (🟢🔴⭐🪜⚠️☠️📋🔍📁🔗🔀🛤️)?
- [ ] Attribute order: `[epic, seq, id]`?
- [ ] SEC-LINK on separate line (not embedded)?
- [ ] SEC-SECTION has closing `!SEC-SECTION`?
- [ ] All `id` values are unique within file?

## Best Practices

- [ ] SEC-SECTION used 5-10 times per file?
- [ ] One SEC-AUDIT per file at top?
- [ ] SEC-SOURCE always has `id` for linking?
- [ ] SEC-BUG has `seq` for severity?
- [ ] SEC-FLOW has `seq` for ordering?
- [ ] SEC-INSTANCE always has `id` attribute?
- [ ] Multiple instances use `seq` for ordering?
- [ ] SEC-RELATION placed before first instance?


## Automated Validation Script

```python
#!/usr/bin/env python3
"""
SCR Framework Compliance Validator
Validates Comment Anchors syntax and coverage
"""

import re
import sys
from pathlib import Path
from collections import defaultdict

class SCRValidator:
    def __init__(self, filepath):
        self.filepath = filepath
        self.content = Path(filepath).read_text()
        self.errors = []
        self.warnings = []
        self.stats = defaultdict(int)
        
    def validate(self):
        """Run all validation checks"""
        self.check_syntax()
        self.check_coverage()
        self.check_metadata_layer()
        self.check_relationships()
        self.check_best_practices()
        return self.report()
    
    def check_syntax(self):
        """Validate syntax compliance"""
        # Find all SEC-* tags
        tags = re.findall(r'//\s*SEC-(\w+)\[(.*?)\]\s*(.*?)$', self.content, re.MULTILINE)
        
        for tag_name, attributes, description in tags:
            self.stats[f'SEC-{tag_name}'] += 1
            
            # Check for epic attribute (except SECTION, LINK)
            if tag_name not in ['SECTION', 'LINK']:
                if 'epic=' not in attributes:
                    self.errors.append(f"SEC-{tag_name} missing 'epic' attribute")
            
            # Check attribute order: epic, seq, id
            if 'epic=' in attributes:
                epic_pos = attributes.find('epic=')
                if 'seq=' in attributes:
                    seq_pos = attributes.find('seq=')
                    if seq_pos < epic_pos:
                        self.warnings.append(f"SEC-{tag_name}: 'seq' before 'epic' (should be epic, seq, id)")
                if 'id=' in attributes:
                    id_pos = attributes.find('id=')
                    if 'seq=' in attributes and id_pos < seq_pos:
                        self.warnings.append(f"SEC-{tag_name}: 'id' before 'seq'")
            
            # Check for emoji
            emojis = ['🟢', '🔴', '⭐', '🪜', '⚠️', '☠️', '📋', '🔍', '📁', '🔗', '🔀', '🛤️']
            if not any(emoji in description for emoji in emojis):
                self.warnings.append(f"SEC-{tag_name} missing emoji in description")
    
    def check_coverage(self):
        """Check layer coverage"""
        # Data Layer
        if self.stats['SEC-SOURCE'] == 0:
            self.warnings.append("No SEC-SOURCE tags found (Data Layer incomplete)")
        if self.stats['SEC-SINK'] == 0:
            self.warnings.append("No SEC-SINK tags found (Data Layer incomplete)")
        
        # Logic Layer
        if self.stats['SEC-LOGIC'] == 0 and self.stats['SEC-FLOW'] == 0:
            self.warnings.append("No SEC-LOGIC or SEC-FLOW tags (Logic Layer unused)")
        
        # Analysis Layer
        if self.stats['SEC-BUG'] == 0:
            self.warnings.append("No SEC-BUG tags found (no findings documented)")
        
        # Organization Layer
        if self.stats['SEC-AUDIT'] == 0:
            self.warnings.append("No SEC-AUDIT tag (missing file context)")
        if self.stats['SEC-AUDIT'] > 1:
            self.warnings.append(f"Multiple SEC-AUDIT tags ({self.stats['SEC-AUDIT']}) - recommend 1 per file")
        
        # Structural Layer
        if self.stats['SEC-SECTION'] == 0:
            self.warnings.append("No SEC-SECTION tags (file not organized)")
        if self.stats['SEC-SECTION'] < 3:
            self.warnings.append(f"Only {self.stats['SEC-SECTION']} SEC-SECTION tags - recommend 5-10 per file")
            
        # Metadata Layer
	    if self.stats['SEC-RELATION'] == 0 and self.stats['SEC-INSTANCE'] == 0:
	        self.warnings.append("No SEC-RELATION or SEC-INSTANCE tags (Metadata Layer unused)")
    
    def check_relationships(self):
        """Check relationship patterns"""
        # Extract all ids
        ids = re.findall(r'id=([a-zA-Z0-9_]+)', self.content)
        id_set = set(ids)
        
        # Check duplicate ids
        if len(ids) != len(id_set):
            duplicates = [id for id in id_set if ids.count(id) > 1]
            self.errors.append(f"Duplicate IDs found: {', '.join(duplicates)}")
        
        # Extract all SEC-LINK targets
        links = re.findall(r'SEC-LINK:.*?#([a-zA-Z0-9_]+)', self.content)
        
        # Check if link targets exist
        for link_target in links:
            if link_target not in id_set:
                self.errors.append(f"SEC-LINK references non-existent ID: #{link_target}")
        
        # Check SEC-SECTION closure
        sections = re.findall(r'SEC-SECTION:', self.content)
        closures = re.findall(r'!SEC-SECTION', self.content)
        if len(sections) != len(closures):
            self.errors.append(f"SEC-SECTION mismatch: {len(sections)} open, {len(closures)} close")
    
    def check_best_practices(self):
        """Check best practice compliance"""
        # Check if SEC-SOURCE has id
        sources = re.findall(r'SEC-SOURCE\[(.*?)\]', self.content)
        for attrs in sources:
            if 'id=' not in attrs:
                self.warnings.append("SEC-SOURCE without 'id' - cannot be linked")
        
        # Check if SEC-BUG has seq (severity)
        bugs = re.findall(r'SEC-BUG\[(.*?)\]', self.content)
        for attrs in bugs:
            if 'seq=' not in attrs:
                self.warnings.append("SEC-BUG without 'seq' - severity not specified")
        
        # Check if SEC-SINK has corresponding SEC-LINK
        sinks = self.content.count('SEC-SINK')
        links_after_sink = len(re.findall(r'SEC-SINK.*?\n.*?SEC-LINK:', self.content, re.DOTALL))
        if sinks > links_after_sink:
            self.warnings.append(f"{sinks - links_after_sink} SEC-SINK tags without SEC-LINK")
        
        # Metadata Layer best practices
	    if self.stats['SEC-RELATION'] > 0 and self.stats['SEC-INSTANCE'] == 0:
	        self.warnings.append("SEC-RELATION found but no SEC-INSTANCE - patterns documented without instances")
    
	    if self.stats['SEC-INSTANCE'] > 0 and self.stats['SEC-RELATION'] == 0:
	        self.warnings.append("SEC-INSTANCE found but no SEC-RELATION - instances without pattern definition")
	    
	def check_metadata_relationships(self):
        """Check metadata layer specific patterns"""
        
        # Check RELATION → INSTANCE pairing
        relations = re.findall(r'SEC-RELATION\[epic=([^,\]]+)', self.content)
        instances = re.findall(r'SEC-INSTANCE\[epic=([^,\]]+)', self.content)
        
        relation_epics = set(relations)
        instance_epics = set(instances)
        
        # Find relations without instances
        for epic in relation_epics:
            if epic not in instance_epics:
                self.warnings.append(f"SEC-RELATION[epic={epic}] has no corresponding SEC-INSTANCE")
        
        # Find instances without relations (less critical)
        for epic in instance_epics:
            if epic not in relation_epics:
                self.warnings.append(f"SEC-INSTANCE[epic={epic}] has no corresponding SEC-RELATION (pattern not documented)")
        
        # Check Composition pattern (INSTANCE → sub-INSTANCEs)
        composition_instances = re.findall(r'SEC-INSTANCE\[epic=Composition.*?id=([^,\]]+)', self.content)
        for parent_id in composition_instances:
            # Check if any other INSTANCE links to this parent
            child_links = re.findall(rf'SEC-LINK:.*?#{parent_id}', self.content)
            if len(child_links) == 0:
                self.warnings.append(f"Composition INSTANCE[id={parent_id}] has no child instances linked to it")
        
        # Check Intersection pattern (multiple RELATIONs)
        intersection_relations = re.findall(r'SEC-RELATION\[epic=Intersection', self.content)
        if len(intersection_relations) > 0:
            # Each intersection should link to 2+ other relations
            for i, match in enumerate(intersection_relations):
                # Find SEC-LINK references after this RELATION
                # Should have 2+ links to other relation IDs
                # This is a simplified check
                pass  # Can be enhanced with more sophisticated parsing
    
    def check_metadata_layer(self):
    """Check metadata layer coverage and patterns"""
    
    # Basic presence check
    if self.stats.get('SEC-RELATION', 0) == 0 and self.stats.get('SEC-INSTANCE', 0) == 0:
        self.warnings.append("Metadata Layer: No SEC-RELATION or SEC-INSTANCE tags found")
        return  # Skip further metadata checks if layer not used
    
    # Extract all RELATION and INSTANCE tags with epics
    relation_pattern = r'SEC-RELATION\[epic=([^,\]]+).*?id=([^,\]]+)?'
    instance_pattern = r'SEC-INSTANCE\[epic=([^,\]]+).*?id=([^,\]]+)?'
    
    relations = re.findall(relation_pattern, self.content)
    instances = re.findall(instance_pattern, self.content)
    
    # Group by epic
    relation_epics = {}
    instance_epics = {}
    
    for epic, rel_id in relations:
        if epic not in relation_epics:
            relation_epics[epic] = []
        relation_epics[epic].append(rel_id if rel_id else None)
    
    for epic, inst_id in instances:
        if epic not in instance_epics:
            instance_epics[epic] = []
        instance_epics[epic].append(inst_id if inst_id else None)
    
    # Rule 2: Pairing check
    for epic in relation_epics:
        if epic not in instance_epics:
            self.warnings.append(f"Metadata Layer: SEC-RELATION[epic={epic}] has no corresponding SEC-INSTANCE")
    
    for epic in instance_epics:
        if epic not in relation_epics:
            self.warnings.append(f"Metadata Layer: SEC-INSTANCE[epic={epic}] exists without SEC-RELATION (pattern not documented)")
    
    # Rule 3: Epic consistency - check against valid relationship types
    valid_epics = [
        'Data_Flow', 'Mirroring', 'Lifecycle', 'Parent', 'Reference',
        'Validation', 'Exploit_Chain', 'Remediation', 'Assumption',
        'AltPath', 'Composition', 'Intersection'
    ]
    
    all_meta_epics = set(list(relation_epics.keys()) + list(instance_epics.keys()))
    for epic in all_meta_epics:
        if epic not in valid_epics:
            self.warnings.append(f"Metadata Layer: Epic '{epic}' is not a standard relationship pattern")
    
    # Rule 6: ID requirement for INSTANCE
    instances_without_id = [epic for epic, ids in instance_epics.items() if None in ids]
    if instances_without_id:
        self.errors.append(f"Metadata Layer: SEC-INSTANCE tags missing required 'id' attribute in epics: {', '.join(instances_without_id)}")
    
    # Rule 4: Composition structure check
    if 'Composition' in instance_epics:
        composition_ids = [id for id in instance_epics['Composition'] if id]
        for parent_id in composition_ids:
            child_links = re.findall(rf'SEC-LINK:.*?#{parent_id}', self.content)
            if len(child_links) == 0:
                self.warnings.append(f"Metadata Layer: Composition INSTANCE[id={parent_id}] has no child instances linked to it")
    
    # Rule 5: Intersection structure check
    if 'Intersection' in relation_epics:
        intersection_ids = [id for id in relation_epics['Intersection'] if id]
        for int_id in intersection_ids:
            # Count SEC-LINK references after this RELATION
            # Should find 2+ links to component relation IDs
            links_after = re.findall(rf'SEC-RELATION\[epic=Intersection.*?id={int_id}.*?\n(.*?)\n.*?SEC-LINK', self.content, re.DOTALL)
            if links_after and len(links_after[0].split('SEC-LINK')) < 3:  # Should have 2+ links
                self.warnings.append(f"Metadata Layer: Intersection RELATION[id={int_id}] should link to 2+ component patterns")
    
    # Rule 7: Sequence usage check
    for epic, ids in instance_epics.items():
        if len(ids) > 1:
            # Check if seq is used
            instances_with_seq = re.findall(rf'SEC-INSTANCE\[epic={epic}.*?seq=', self.content)
            if len(instances_with_seq) == 0:
                self.warnings.append(f"Metadata Layer: Multiple SEC-INSTANCE[epic={epic}] found but no 'seq' attribute for ordering")
    
    def report(self):
        """Generate validation report"""
        print(f"\n{'='*60}")
        print(f"SCR Framework Validation Report: {self.filepath}")
        print(f"{'='*60}\n")
        
        # Statistics
        print("Tag Statistics:")
        for tag, count in sorted(self.stats.items()):
            print(f"  {tag}: {count}")
        print()
        
        # Errors
        if self.errors:
            print(f"❌ ERRORS ({len(self.errors)}):")
            for error in self.errors:
                print(f"  • {error}")
            print()
        
        # Warnings
        if self.warnings:
            print(f"⚠️  WARNINGS ({len(self.warnings)}):")
            for warning in self.warnings:
                print(f"  • {warning}")
            print()
        
        # Summary
        if not self.errors and not self.warnings:
            print("✅ All checks passed!")
            return 0
        elif not self.errors:
            print(f"✅ No errors, {len(self.warnings)} warnings")
            return 0
        else:
            print(f"❌ {len(self.errors)} errors, {len(self.warnings)} warnings")
            return 1

if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Usage: scr_validate.py <file>")
        sys.exit(1)
    
    validator = SCRValidator(sys.argv[1])
    sys.exit(validator.validate())
```
