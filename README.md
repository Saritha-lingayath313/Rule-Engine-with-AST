# Rule-Engine-with-AST
1. Data Structure for the AST (Abstract Syntax Tree)
We will represent the rules using a tree-like structure where each node is either an operator (AND, OR) or an operand (conditions like age > 30 or department = 'Sales').

Here’s the data structure for the AST:


class Node:
    def init(self, node_type, left=None, right=None, value=None):
        self.node_type = node_type  # "operator" or "operand"
        self.left = left            # left child
        self.right = right          # right child
        self.value = value          # condition value for operands
2. Creating the AST from Rule Strings
We’ll use a recursive function to parse the rule strings and create an AST.


import ast
import operator

def create_rule(rule_string):
    # Parse the rule string into a Python AST tree using ast.parse
    parsed_rule = ast.parse(rule_string, mode='eval')
    return build_ast(parsed_rule.body)

def build_ast(tree_node):
    if isinstance(tree_node, ast.BoolOp):  # AND/OR operator
        node = Node('operator', value=type(tree_node.op).name)
        node.left = build_ast(tree_node.values[0])
        node.right = build_ast(tree_node.values[1])
    elif isinstance(tree_node, ast.Compare):  # Comparison operand (e.g., age > 30)
        left_value = tree_node.left.id if isinstance(tree_node.left, ast.Name) else tree_node.left.n
        op = type(tree_node.ops[0]).name
        right_value = tree_node.comparators[0].n if isinstance(tree_node.comparators[0], ast.Num) else tree_node.comparators[0].s
        condition = f"{left_value} {op} {right_value}"
        node = Node('operand', value=condition)
    return node
3. Combining Multiple Rules
To combine rules efficiently, we’ll merge the ASTs of the rules with the AND or OR operators.


def combine_rules(rule_nodes):
    if not rule_nodes:
        return None
    # Combine rules with AND operator for simplicity
    combined_node = rule_nodes[0]
    for rule in rule_nodes[1:]:
        combined_node = Node('operator', left=combined_node, right=rule, value='And')
    return combined_node
4. Evaluating the AST Against Data
We’ll evaluate the AST by recursively traversing the tree and applying the rules to the provided data.

def evaluate_rule(rule_node, data):
    if rule_node.node_type == 'operand':
        return eval_operand(rule_node.value, data)
    elif rule_node.node_type == 'operator':
        if rule_node.value == 'And':
            return evaluate_rule(rule_node.left, data) and evaluate_rule(rule_node.right, data)
        elif rule_node.value == 'Or':
            return evaluate_rule(rule_node.left, data) or evaluate_rule(rule_node.right, data)
    return False

def eval_operand(condition, data):
    # Evaluate conditions like 'age > 30' or 'department == "Sales"'
    try:
        return eval(condition, {}, data)
    except Exception as e:
        return False
5. Sample Database Schema for Storing Rules
You could use a SQL or NoSQL database. For simplicity, let’s go with SQL and assume the following schema:

sql
Copy code
CREATE TABLE rules (
    id INT PRIMARY KEY AUTO_INCREMENT,
    rule_string TEXT NOT NULL,
    rule_ast JSON NOT NULL
);
6. API Functions
Here's how you would implement the API functions:

a. create_rule(rule_string)
This function would call the create_rule method described earlier and store the AST in the database.


def create_rule_api(rule_string):
    ast_tree = create_rule(rule_string)
    # Convert AST to JSON and store in DB (pseudo-code)
    # db.insert('rules', {'rule_string': rule_string, 'rule_ast': json.dumps(ast_tree)})
    return ast_tree
b. combine_rules(rules)
This function will combine multiple ASTs into one, as explained earlier.


def combine_rules_api(rules):
    rule_nodes = [create_rule(rule_string) for rule_string in rules]
    combined_ast = combine_rules(rule_nodes)
    return combined_ast
c. evaluate_rule(JSON data)
This function will evaluate the rule against the provided user data.




      
So, honestly this was a good question. Had to use my knowledge of compilers, algorithmic knowledge
and software engineering principles. It hit all 3 important pillars, academics, DSA and Dev.
It felt great to do this assignment, I also got to learn a lot about the heuristics in this space,
and I am thankful for it.
