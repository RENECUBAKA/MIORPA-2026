# MIORPA-2026

AnalyzeGroups := function(start_n, end_n, filename)
    local output, header, groups, i, G, group_id, center_size, classes, num_classes,
          sizes, sorted_sizes, size_set, rank,
          class_size_sum, element_count, ratio, nilpotent, solvable, derived_length,
          fitting_subgroup, fitting_derived_length, fitting_size,
          prime_divisors, prime_string,
          commuting_prob, result, row, n, total_groups, processed_groups,
          row_data, row_strings;

    # Check and load SmallGrps package
    if not IsPackageMarkedForLoading("SmallGrp", "") then
        Print("Loading SmallGrps package...\n");
        if LoadPackage("SmallGrp") = fail then
            Print("ERROR: SmallGroups package could not be loaded.\n");
            Print("Please install the SmallGroups package first.\n");
            return fail;
        fi;
    fi;

    # Validate input parameters
    if not IsInt(start_n) or not IsInt(end_n) or start_n < 1 or end_n < start_n then
        Error("Inputs must be positive integers with start_n <= end_n");
    fi;

    # Set default filename if not provided
    if filename = fail or filename = "" then
        filename := Concatenation("Groups_of_order_", String(start_n), "_to_", String(end_n), ".csv");
    fi;

    # Initialize output
    output := [];
    header := "Order;GroupID;CenterSize;kG;ClassSizes;Rank;Nilpotent;Solvable;DerivedLength;FittingDerivedLength;FittingSize;PrimeDivisors;CommutingProb;Ratio\n";
    Append(output, header);

    total_groups := 0;
    processed_groups := 0;

    # Loop through all orders
    for n in [start_n..end_n] do
        groups := AllSmallGroups(n);
        total_groups := total_groups + Length(groups);
        
        # If no groups of this order, skip
        if Length(groups) = 0 then
            Print("No groups of order ", n, "\n");
            continue;
        fi;
        
        # Process each group of this order
        for i in [1..Length(groups)] do
            G := groups[i];

            # Group ID
            group_id := Concatenation(String(n), ".", String(i));

            # Center size
            center_size := Size(Center(G));

            # Conjugacy classes
            classes := ConjugacyClasses(G);
            num_classes := Length(classes);

            # Extract and sort class sizes
            sizes := List(classes, Size);
            sorted_sizes := SortedList(sizes);
            rank := Length(Set(sorted_sizes));
            
            # Calculate ratio (order / number of classes)
            if num_classes > 0 then
                ratio := Float(n / num_classes);
            else
                ratio := 0;
            fi;

            # Group properties
            nilpotent := IsNilpotent(G);
            solvable := IsSolvable(G);
            derived_length := DerivedLength(G);

            # Fitting subgroup
            fitting_subgroup := FittingSubgroup(G);
            if IsTrivial(fitting_subgroup) then
                fitting_derived_length := 0;
                fitting_size := 1;
            else
                fitting_derived_length := DerivedLength(fitting_subgroup);
                fitting_size := Size(fitting_subgroup);
            fi;

            # Prime divisors
            prime_divisors := PrimeDivisors(n);

            # Commuting probability
           
             commuting_prob := Float(num_classes) / Float(n);


            # Prepare row data with proper types
            row_data := [
                n,                                    # Order
                group_id,                             # GroupID
                center_size,                          # CenterSize
                num_classes,                          # kG
                sorted_sizes,                         # ClassSizes
                rank,                                 # Rank
                nilpotent,                            # Nilpotent
                solvable,                             # Solvable
                derived_length,                       # DerivedLength
                fitting_derived_length,               # FittingDerivedLength
                fitting_size,                         # FittingSize
                prime_divisors,                       # PrimeDivisors
                commuting_prob,                       # CommutingProb
                ratio                                 # Ratio
            ];

            # Convert all row data to strings for CSV
            row_strings := List(row_data, function(x)
                if IsString(x) then
                    return x;
                elif IsList(x) then
                    # Handle lists properly - remove spaces for cleaner CSV
                    return ReplacedString(String(x), " ", "");
                else
                    return String(x);
                fi;
            end);

            # Create CSV row
            row := JoinStringsWithSeparator(row_strings, ";");
            Append(output, row);
            Append(output, "\n");

            processed_groups := processed_groups + 1;

            # Print progress every 100 groups
            if processed_groups mod 100 = 0 then
                Print("Processed ", processed_groups, " groups so far...\n");
            fi;
        od;
    od;

    # Write output to file
    PrintTo(filename, output);

    Print("\n========================================\n");
    Print("Done! Processed ", total_groups, " groups of orders ", start_n, " to ", end_n, "\n");
    Print("Results saved to: ", filename, "\n");
    Print("========================================\n");

    return filename;
end;
